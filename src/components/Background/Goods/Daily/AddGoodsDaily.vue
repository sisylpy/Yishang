<template>

    <!-- model-applycommit-->

    <div class="my-modal-back" id="modal_daily">
        <div class="my-modal">
            <div class="">
                <div class="">
                    <button type="button" class="close" @click="close"  data-dismiss="modal" aria-label="Close">
                        <span aria-hidden="true">&times;</span></button>
                    <h4 class="modal-title" id="memberWxNameApply">{{fatherName}}</h4>
                </div>
                <div class="my_modal_body">
                    <!--<p>{{fatherName}}</p>-->
                    <form class="form-horizontal">
                        <div class="form-group">
                            <div class="col-sm-2 control-label">商品类别名称</div>
                            <div class="col-sm-10">
                                <input type="text" class="form-control" v-model="ckGoods.goodsName" placeholder="商品名称"
                                       value="name"/>
                                <input type="hidden"  :name="ckGoods.fatherId" :v-model="fatherId" >
                            </div>
                        </div>

                        <div class="form-group">
                            <div class="col-sm-2 control-label">采购规格</div>
                            <div class="col-sm-10">
                                <input type="text" class="form-control" v-model="ckGoods.purStandardName"
                                       placeholder="规格"/>
                            </div>
                        </div>
                        <div class="form-group">
                            <div class="col-sm-2 control-label">申请规格</div>
                            <div class="col-sm-10">
                                <input type="text" class="form-control" v-model="ckGoods.applyStandardName"
                                       placeholder="申请规格"/>
                            </div>
                        </div>

                        <div class="form-group">
                            <div class="col-sm-2 control-label">所属大类</div>
                            <div class="col-sm-10">
                                <select class="form-control" v-model="ckGoods.fatherId">
                                    <option :value="item.goodsId" v-for="item in fatherArr">{{item.goodsName}}</option>
                                </select>
                            </div>
                        </div>

                        <div class="form-group">
                            <div class="col-sm-2 control-label">出货部门</div>
                            <div class="col-sm-10">
                                <select class="form-control" v-model="ckGoods.outDepId" @change="selectOutDep">
                                    <option :value="item.depId" v-for="item in outDepArr">{{item.depName}}</option>
                                </select>
                            </div>
                        </div>


                        <div class="form-group">
                            <div class="col-sm-2 control-label">零售价格</div>
                            <div class="col-sm-10">
                                <input class="form-control" v-model="ckGoods.price"/>
                            </div>
                        </div>

                    </form>

                </div>
                <div class="my-modal-footer">
                    <button type="button" class="btn btn-sm btn-default" @click="close">关闭</button>
                    <button type="button" class="btn btn-sm btn-primary"  @click="saveOrUpdate">确定</button>
                </div>
            </div>
            <!-- /.modal-content -->
        </div>
        <!-- /.modal-dialog -->
    </div>


</template>

<script>
    import api from '@/api/background/goods'
    import apio from '@/api/background/ckDep'

    export default {
        name: "AddGoodsDaily",
        data() {
            return {

                title: "新增",
                ckGoods: {
                    outDepId: '',
                    type: 2

                },
                selectedStatus: 1,
                outDepArr: [],
                fatherArr:[],

            }
        },
        props:['fatherId','fatherName','goodsType',  'addGoods'],
        mounted(){
            var that = this;

            // $('#modal_inventory').on('show.bs.modal', function (e) {
            //     that.fatherName = $(e.relatedTarget).attr('fathername');
            //     that.fatherId = $(e.relatedTarget).attr('fatherid');
            //
            // })

        },
        watch: {
            fatherId: function(newVal,oldVal){
                this.fatherId = newVal
                this.ckGoods.fatherId = newVal;

            },
            fatherName: function (newVal, oldVal) {

            },
            goodsType: function (newVal, oldVal) {
                this.goodsType = newVal
                this.ckGoods.type = newVal;
                this.getFatherList();


            },
            addGoods: function (newVal, oldVal) {
                console.log(newVal);

                if(newVal == 1) {
                    console.log("shi new dkfalsjldasj;lkyeayayyay")
                    var goodsId = $('#modal_daily').attr('goodsid');

                    console.log(goodsId)
                    if ( goodsId!= null) {
                        this.goodsId = goodsId
                        this.title = "修改";
                        this.getInfo(goodsId);
                    }
                }
            }

        },
        created: function () {

            if (this.goodsId != null) {
                this.title = "修改";
                this.getInfo(this.goodsId);
            }
            this.getOutDepData();



        },
        components: {},

        methods: {
            getFatherList: function () {
                api.queryTypeFatherList(2).then(res => {
                    if (res) {
                        this.fatherArr = res.data;
                    }
                })
            },
            close:function(){
                $('#modal_daily').hide();
                this.goodsId = null;
                this.ckGoods = {};
                this.$emit('submit-add');

            },
            selectOutDep: function (e) {
                console.log(this.ckGoods.outDepId);

            },
            getOutDepData: function () {
                console.log("get le 3ci  deplist???")

                var type = 1;
                apio.getOutDepList(type).then(res => {
                    console.log(res);

                    if (res) {
                        this.outDepArr = res.data
                    }
                })
            },
            getInfo: function () {
                console.log("huoqu api???");

                console.log(this.goodsId);

                api.getGoodsInfo(this.goodsId).then(res => {
                    this.ckGoods = res.data;
                    console.log(res);

                    $('#goodsStatus option[value= "' + this.ckGoods.status + '"]').attr("selected", true);   //设置Select的Text值为jQuery的项选中
                    $('#goodsStatus option[value= "' + this.ckGoods.isWeight + '"]').attr("selected", true);   //设置Select的Text值为jQuery的项选中

                })
            },
            saveOrUpdate: function (event) {
                console.log("saveOrUpdate api ")

                if (this.ckGoods.goodsId) {

                    api.updateGoods(this.ckGoods).then(res => {
                        $('#modal_daily').hide();
                        this.$emit('submit-add');
                        this.goodsId = null;
                        this.ckGoods = {};


                    })
                } else {


                    console.log(this.ckGoods);

                    api.saveGoods(this.ckGoods).then(res => {
                        if(res.code == 0) {
                            $('#modal_daily').hide();
                            //
                            this.$emit('submit-add');
                            this.goodsId = null;
                            this.ckGoods = {};
                        }


                    })
                }

            },

            // back: function () {
            //     $('#modal-warning').modal('hide')
            //
            //     this.$router.go(-1)
            // }
        }
    }
</script>

<style>


</style>
