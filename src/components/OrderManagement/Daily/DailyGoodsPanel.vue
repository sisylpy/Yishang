<template>

    <div class="my_panel">

        <div class="row my-drop-group">


            <div class="my-dropDown-group col-md-6">
                <h5>负责订货商品的称重拣货</h5>
                <div class="form-group"  style="background: yellow">
                    <select class="form-control select2"  data-placeholder="出库部门"
                            style="width: 100%;" id="selectOutDepDaily" >
                        <option ></option>
                        <option v-for="(item) in outDepArr" :value="item.depId"> {{item.depName}}</option>
                    </select>

                </div>

            </div>

            <div class="my-dropDown-group col-md-6">
                <h5>商品类别</h5>
                <div class="form-group"  style="background: yellow">
                    <select class="form-control select2"  data-placeholder="类别"
                            style="width: 100%;" id="selectFatherDaily" >
                        <option ></option>
                        <option v-for="(item) in cateArr" :value="item.goodsId" > {{item.goodsName}}</option>
                    </select>

                </div>
            </div>


        </div>




        <div class="panel panel-default">
            <div class="panel-body  no-border no-padding">
                <div class="box-body  no-padding">

                    <table id="jqGridDailyPanel"></table>
                    <div id="jqGridPagerDaily"></div>

                </div>

            </div>

        </div>


    </div>




</template>

<script>
    import api from '@/api/orderManagement/inventory'
    import apig from '@/api/background/goods'

    export default {
        name: "DailyGoodsPanel",
        props:['dailyType'],
        watch: {

            dailyType: function (newVal) {
                if (newVal == "dailyGoods") {
                    this.getSortsList()
                    this.getJqtableData();
                }
            }

        },

        data() {
            return {
                page: 1,
                limit: 200,
                goodsType: 2,
                fatherId: -1,
                depId: -1,
                goodsList:[],
                outDepArr: [],
                cateArr: [],
            }
        },

        mounted(){
            var that = this;

            $('.select2').select2();

            // selcct 产品
            $('#selectOutDepDaily').on('change', function (e) {

                that.depId = $(this).val();
                that.getJqtableData();
            });

            $('#selectFatherDaily').on('change', function (e) {

                console.log("selfatheer")
                that.fatherId = $(this).val();
                that.getJqtableData();
            });
            // this.getSortsList()
            this.getJqtableData();


            //修改采购数量
            $('#jqGridDailyPanel').on('dblclick', 'input[name=plan]' ,function (e) {
                console.log("doubleleleleele")
                $(this).removeAttr("disabled");
                $(this).focus();


            })

            $('#jqGridDailyPanel').on('keyup', 'input[name=plan]', function (e) {

                if (e.keyCode == 13) {
                    // var newPrice = $(this).val();
                    console.log(e.currentTarget.id)


                    var goodsId = e.currentTarget.id;
                    var goods = {
                        goodsId:  goodsId,
                        planPurchase: $(this).val(),
                    }

                    apig.updateGoods(goods).then(res =>{
                        if(res.code == 0){
                            that.getJqtableData();

                            $(e.currentTarget).blur();
                            $(e.currentTarget).attr("disabled")

                        }
                    })


                }
            });


        },
        methods: {



            //
            // queryFatherGoods: function(event) {
            //     var $this = $(event.currentTarget);
            //     this.fatherId = $this.attr('fatherId');
            //     this.fatherName = $this.html();
            //     this.getJqtableData(this.depId, this.fatherId)
            //     $('#dropdownMenu1').html( this.fatherName+`<span class="caret"></span>`)
            //
            // },

            getSortsList: function () {
                // this.bus.$emit('loading', true);

                api.getOutDepAndCate(this.goodsType).then(res => {
                    if(res) {
                        console.log(res)
                        this.cateArr = res.data.fatherList;
                        this.outDepArr = res.data.outDepList;
                        // this.getJqtableData();
                        // this.bus.$emit('loading', false);


                    }
                })

            },

            //获取表格数据
            getJqtableData: function(){
                var data = "page=" + this.page + "&limit=" + this.limit  +
                    "&depId="+ this.depId + "&fatherId=" + this.fatherId + "&type="+ this.goodsType;
                this.bus.$emit('loading', true);

                api.getOutDepTypeGoodsList(data).then(res => {
                    this.goodsList = res.page.list;
                    console.log(res.page)
                    //加载表格数据
                    this.jqtable()
                    this.bus.$emit('loading', false);
                    this.getSortsList();

                });

            },

            // 初始化表格
            jqtable() {

                console.log("jqtable?????")

                // 清空jqGrid表格数据
                $("#jqGridDailyPanel").jqGrid("clearGridData")

                // 初始化jqgrid
                var _this = this

                //更新数据
                $("#jqGridDailyPanel").jqGrid('setGridParam',{
                    datatype:'local',
                    data:this.goodsList,//newData是符合格式要求的重新加载的数据
                    page:this.currPage//哪一页的值
                }).trigger("reloadGrid");

                $("#jqGridDailyPanel").jqGrid('setLabel', '0', '序号', 'labelstyle');

                $("#jqGridDailyPanel").jqGrid(
                    {
                        data: _this.goodsList,
                        datatype: "local",
                        colModel: [
                            {label: 'goodsId', name: 'goodsId', width: 50, key: true, hidden: true},
                            {label: '产品名称', name: 'goodsName', width: 120},
                            {label: '出货部门', name: 'goodsName', width: 120, formatter: function(value, options, row){
                                    return name = row.outDepEntity.depName
                                }},


                            {label: '今日订货', name: 'todayQuantity', width: 120, formatter: function(value, options, row){
                                    return name = value + row.purStandardName
                                }},
                            {label: '采购数量', name: 'planPurchase', width: 120, formatter: function(value, options, row){
                                if(value){
                                    return "<input style='width: 60%' disabled id="+row.goodsId+" value=  "+value+" name='plan'/>"+row.purStandardName+"";
                                }else {
                                    return "<input style='width: 60%' disabled id="+row.goodsId+" placeholder='0' name='plan'/>" + row.purStandardName+" "
                                }
                                }}

                        ],


                        viewrecords: true,
                        height: 440,
                        rowNum: 10,
                        rowList: [10, 30, 50],
                        rownumbers: true,
                        rownumWidth: 25,
                        shrinkToFit:false,
                        autowidth: true,
                        autoScroll: true,
                        multiselect: false,
                        pager: "#jqGridPagerDaily",
                        jsonReader: {
                            root: "page.list",
                            page: this.currPage,
                            total: this.totalCount,
                            records: this.totalPage,
                        },
                        prmNames: {
                            page: "page",
                            rows: "limit",
                            order: "order"
                        },
                        // gridComplete: function () {
                        //     //隐藏grid底部滚动条
                        //     $("#jqGrid").closest(".ui-jqgrid-bdiv").css({"overflow-x": "scroll"});
                        // }
                    });

            },




        }


    }

</script>

<style scoped>
    /*.content{*/
    /*background: #fff;*/
    /*}*/




    .panel-title {
        height: 40px;
    }

    .my-warning{
        /*background: #007a5a;*/
        background: #e07e20;
        color: #fff;
    }



</style>
