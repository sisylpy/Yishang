<template>


    <div>
        <div class="content container-fluid box box-default no-padding" id="jqbody">
            <div class="box-header with-border">
                <h3 class="box-title" style="margin-left: 10px;">{{tableName}}</h3>

                <div class="box-tools pull-left">

                    <div class="btn-group">
                        <button type="button" class="btn btn-box-tool dropdown-toggle" data-toggle="dropdown">
                            <i class="fa fa-wrench"></i></button>
                        <ul class="dropdown-menu" role="menu">

                            <li><a   @click="add">新增商品</a></li>
                            <li><a @click="update">修改商品</a></li>
                            <li><a  @click="del">删除商品</a></li>
                            <li class="divider"></li>
                            <li><a @click="delOne">新增商品大类</a></li>
                            <li><a @click="delOne">修改商品大类</a></li>
                            <li><a @click="delOne">删除商品大类</a></li>
                            <li class="divider"></li>
                            <li><a >导入</a></li>
                            <li><a >导出</a></li>
                        </ul>
                    </div>
                </div>

            </div>

            <div class="box-body no-padding">
                <table id="jqGridInventory"></table>
                <div id="jqGridPagerInventory"></div>
            </div>

            <AddGoods :fatherName="fatherName" :fatherId="fatherId" :goodsType="goodsType" :addGoods="addGoods" @submit-add="appendToList "/>

        </div>

    </div>

</template>

<script>
    import api from '../../../../api/background/goods'
    import AddGoods from '@/components/Background/Goods/Inventory/AddGoodsInventory.vue'

    export default {

        name: "InventoryTypeGoodsTable",
        props:['fatherId','fatherName','goodsType'],
        components:{
            AddGoods
        },

        watch: {
            fatherId: function(newVal,oldVal){
                this.fatherId = newVal
                this.getJqtableData(newVal)

            },
            fatherName: function (newVal, oldVal) {
                this.tableName= newVal;

            },
            goodsType: function (newVal, oldVal) {
                if(newVal == "stock"){
                    this.goodsType = newVal;
                    this.getJqtableData(this.fatherId);
                }
            },
        },
        data() {
            return {
                page: 1,
                limit: 100,
                goodsList: [],
                tableName: "没有数据，请刷新页面",
                addGoods: 0,
                fatherArr:[]
            }
        },



        methods: {


            delOne:function(){
                if(this.goodsList.length == 0){
                    console.log("===000")
                    api.deleteOne(this.fatherId).then(res => {
                        if(res.code == 0){
                            this.$emit('del-father');
                        }
                    })
                }
            },



            appendToList: function() {
                this.editGoodsId = "-1";
                this.getJqtableData(this.fatherId)
            },

            //获取表格数据
            getJqtableData: function(newVal){
                console.log("laigengxinshujua!!!!")
                var data = "page=" + this.page + "&limit=" + this.limit + "&fatherId=" + newVal;
                api.getCateGoodsList(data).then(res => {
                    console.log(data);
                    console.log("sisy");
                    console.log(res.page.list);

                    this.goodsList = res.page.list;
                    //加载表格数据
                    this.jqtable()
                });
            },


            // 初始化表格
            jqtable() {

                // 清空jqGrid表格数据
                $("#jqGridInventory").jqGrid("clearGridData")

               // 初始化jqgrid
                var _this = this

                //更新数据
                $("#jqGridInventory").jqGrid('setGridParam',{
                    datatype:'local',
                    data:this.goodsList,//newData是符合格式要求的重新加载的数据
                    page:this.currPage//哪一页的值
                }).trigger("reloadGrid");

                $("#jqGridInventory").jqGrid('setLabel', '0', '序号', 'labelstyle');

                $("#jqGridInventory").jqGrid(
                    {
                        data: _this.goodsList,
                        datatype: "local",
                        colModel: [
                            {label: 'goodsId', name: 'goodsId', width: 50, key: true, hidden: true},
                            {label: '产品名称', name: 'goodsName', width: 120},
                            {label: '采购规格', name: 'purStandardName', width: 80},
                            {label: '申请规格', name: 'applyStandardName', width: 80},
                            {label: '出货部门', name: 'outDepEntity.depName', width: 80},
                            {label: '采购部门', name: 'purDepEntity.depName', width: 80},
                            {label: '零售价格', name: 'price', width: 80},
                            {label: '拼音', name: 'pinyin', width: 120},
                            {label: '简拼', name: 'headPinyin', width: 80},

                            {label: '库存', name: 'stockPurStandard', width: 110,
                                formatter: function (value, options, rowData) {
                                    return name = rowData.stockPurStandard + rowData.purStandardName
                                }},

                        ],


                        viewrecords: true,
                        height: 400,
                        rowNum: 10,
                        rowList: [10, 30, 50],
                        rownumbers: true,
                        rownumWidth: 25,
                        shrinkToFit:false,
                        autowidth: true,
                        autoScroll: true,
                        multiselect: true,
                        pager: "#jqGridPagerInventory",
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


            add: function () {

                $('#modal_inventory').show();

            },

            update: function (event) {
                var goodsId = this.getSelectedRow();
                this.addGoods = 1;
                if (goodsId == null) {
                    return;
                }

                $('#modal_inventory').show();
                $('#modal_inventory').attr('goodsid', goodsId);


            },


            del: function (event) {
                var goodsIds = this.getSelectedRows();
                if (goodsIds == null) {
                    return;
                }

                api.deleteGoods(goodsIds).then(res => {
                    if(res) {
                        this.appendToList();
                        // this.$router.go(-1)
                        // $('#modal-warning').modal('hide')

                    }
                })
                var paramContent = this.getSelectedRowsContents("goodsName");

                // this.$router.push({
                //     name: '/op_assignGoods/madal_warning',
                //     params: {
                //         modal_title: "确定要删除" + paramContent + "吗？",
                //         goodsIds: goodsIds
                //     }
                // })

            },


            delGoods: function () {

            },
            delFinished: function () {
                console.log("delFinish");
                this.jqtable()

            },

            //选择一条记录
            getSelectedRow: function () {
                var grid = $("#jqGridInventory");
                var rowKey = grid.getGridParam("selrow");
                if (!rowKey) {
                    console.log("请选择一条记录a");
                    return;
                }

                var selectedIDs = grid.getGridParam("selarrrow");

                if (selectedIDs.length > 1) {

                    console.log("只能选择一条记录b");
                    return;
                }
                // var celldata = $("jqgridtableid").jqGrid('getCell',id,colnum);
                return selectedIDs[0];
            },




            //选择多条记录
            getSelectedRows: function () {
                var grid = $("#jqGridInventory");
                var rowKey = grid.getGridParam("selrow");
                if (!rowKey) {
                    alert("请选择一条记录");
                    return;
                }
                return grid.getGridParam("selarrrow");
            },

            //选中多条数据内容
            getSelectedRowsContents: function (param) {

                var ids = this.getSelectedRows();

                var grid = $("#jqGridInventory");
                var contents = [];

                for (var i = 0; i < ids.length; i++) {
                    var paramContent = grid.jqGrid('getRowData', ids[i])[param];
                    contents.push(paramContent);
                }
                return contents;
            },

            // getSelectRowParm: function (param) {
            //     var grid = $("#jqGrid");
            //     var paramContent = grid.jqGrid('getRowData', grid.getGridParam("selarrrow")[0]).param;
            //
            //     return paramContent;
            // },



        }


    }

</script>

<style >
    /*.content{*/
        /*background: #fff;*/
    /*}*/


    .my-modal-back{
        position: fixed;
        top: 0;
        right: 0;
        bottom: 0;
        left: 0;
        background-color: rgba(0,0,0,.5);
        width: 100%;
        height: 100%;
        /*display: flex;*/
        justify-content: center;
        align-items: center;
        z-index: 99999999;
        display: none;

    }
    .my-modal {
        position: relative;
        width: 50%;
        margin-left: 25%;

        background: #fff;
        padding: 20px;
        /*color: #fff;*/
        margin-top: 50px;


    }



    .my_modal_body {
        /*padding: 130px;*/


    }
    .my-modal-footer{
        display: flex;
        flex-flow: row nowrap;
        justify-content: flex-end;
        margin-right: 40px;
        align-items: center;
    }


</style>
