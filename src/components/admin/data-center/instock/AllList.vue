<template>
  <admin-nav></admin-nav>
  <div class="container-fluid">
    <div class="row">
      <left-datacenter></left-datacenter>
      <div class="col-lg-10">
        <!-- 路径导航 -->
        <ol class="breadcrumb breadcrumbs">
          <li><a v-link="{ path: '/admin/dataCenter/instock/checkList'}"> 盘点单统计</a></li>
          <li><a v-link="{ path: '/admin/dataCenter/instock/allocateList'}"> 调拨单统计</a></li>
          <li><a v-link="{ path: '/admin/dataCenter/instock/requisitionList'}"> 要货单统计</a></li>
          <li><a v-link="{ path: '/admin/dataCenter/instock/distributionList'}"> 配送单统计</a></li>
          <li class="active"><a v-link="{ path: '/admin/dataCenter/instock/allList'}"> 收发存汇总表</a></li>
          <li><a v-link="{ path: '/admin/dataCenter/instock/remainderList'}"> 存货总账</a></li>
          <li><a v-link="{ path: '/admin/dataCenter/instock/orderSaleList'}"> 零售出库单统计</a></li>
        </ol>
        <!-- 页头 -->
        <div class="page-header">
          <form class="form-inline">
            <div class="form-group">
              <label>仓库</label>
              <select class="form-control" v-model="searchData.warehouse_id">
                <option value="">请选择</option>
                <option :value="item.id" v-for="item in providerList">{{item.name}}</option>
              </select>
            </div>
            <div class="form-group  ml10">
              <label>货号</label>
              <input type="text" class="form-control" placeholder="请输入货号" v-model="searchData.item_code">
            </div>
            <div class="form-group  ml10">
              <label>品名</label>
              <input type="text" class="form-control" placeholder="请输入商品名称" v-model="searchData.item_name">
            </div>
            <div class="form-group ml10">
              <label>时间段</label>
              <date-picker :value.sync="searchData.start_time" time-text=开始时间></date-picker> -
              <date-picker :value.sync="searchData.end_time"  time-text=结束时间></date-picker>
            </div>
            <span class="btn btn-primary " @click="searchMethod(1)">搜索</span>
            <span class="btn btn-warning" @click="searchCancel()">撤销搜索</span>

            <a :href="exports" target="_blank"><span class="btn btn-info spanblocks fr mr10">导出</span></a>
            <!--<a v-link="{ path: '/admin/dataCenter/instock/remainderListDetail'}" class="btn btn-primary spanblocks fr mr10">明细</a>-->
          </form>
        </div>

        <!-- 统计表格 -->
        <table class="table table-striped table-border table-hover mt20" :data="onedata">
          <tbody>
          <tr>
            <td>
              <span class="pr30">期初库存汇总：{{onedata.start_stock}}</span>
              <span class="pr30">期初库存总额：{{onedata.start_stock_total_sum}}</span>
              <span class="pr30">期间入库汇总：{{onedata.in_stock}}</span>
              <span class="pr30">期间入库汇总额：{{onedata.in_stock_total_sum}}</span>
              <span class="pr30">期间出库汇总：{{onedata.out_stock}}</span>
              <span class="pr30">期间出库汇总额：{{onedata.out_stock_total_sum}}</span>
              <span class="pr30">期末库存汇总：{{onedata.current_stock}}</span>
              <span class="pr30">期末库存总额：{{onedata.current_stock_total_sum}}</span>
            </td>
          </tr>
          </tbody>
        </table>

        <!-- 列表渲染表格 -->
        <grid :data="listdata" :columns="gridColumns" :operate="productOperate"></grid>

        <!--翻页-->
        <page :total='page.total' :current.sync='page.current_page' :display='page.per_page' :last-page='page.last_page' v-if="listdata.length>0"></page>

      </div>
    </div>
  </div>
  <!--错误信息-->
  <error-tip :err-modal.sync="modal.errModal" :err-info="modal.errInfo"></error-tip>
</template>
<style>

</style>
<script>
  import $ from 'jquery'
  import Grid from '../../../common/Grid'
  import Page from '../../../common/Page'
  import AdminNav from '../../AdminNav'
  import Modal from '../../../common/Modal'
  import Summary from '../../../common/Summary'
  import DatePicker from  '../../../common/DatePicker'
  import ErrorTip from '../../../common/ErrorTip'
  import LeftDatacenter from '../../common/LeftDataCenters'
  import {requestUrl,requestSystemUrl,getDataFromApi,token,exchangeData,searchRequest,deleteRequest,checkRequest,finishRequest} from '../../../../publicFunction/index'
  export default{
    components: {
      Grid: Grid,
      Page: Page,
      Modal: Modal,
      AdminNav: AdminNav,
      Summary: Summary,
      DatePicker: DatePicker,
      ErrorTip: ErrorTip,
      LeftDatacenter: LeftDatacenter
    },
    events: {
//    绑定翻页事件
      pagechange: function (currentpage) {
        this.getListData(currentpage)
      }
    },
    ready: function () {
//      获取列表数据
      this.getListData(1)
//      获取单条汇总数据
      this.getOneData(1)
//      获取仓库列表
      this.getProviderList()
    },
    methods: {
//      获取数据列表
      getListData: function(page){
        var self = this
        var url = requestSystemUrl + '/backend-system/data-center/stock/log-list'
        var data = {
          start_time: self.searchData.start_time,
          end_time: self.searchData.end_time,
          item_code: self.searchData.item_code,
          item_name: self.searchData.item_name,
          warehouse_id: self.searchData.warehouse_id,
          page: page
        }
        getDataFromApi(url,data,function(response){
          self.listdata = response.data.body.list
          self.page = response.data.body.pagination
          self.modifyGetedData(self.listdata)
        },function(err){})
      },
//      获取单条数据
      getOneData: function(page){
        var self = this
        var url = requestSystemUrl + '/backend-system/data-center/stock/log-list/total'
        var data = {
          start_time: self.searchData.start_time,
          end_time: self.searchData.end_time,
          item_code: self.searchData.item_code,
          item_name: self.searchData.item_name,
          warehouse_id: self.searchData.warehouse_id
        }
        getDataFromApi(url,data,function(response){
          self.onedata = response.data.body.list
          self.onedata.start_stock_total_sum ='￥' +  (self.onedata.start_stock_total_sum * 0.01).toFixed(2)
          self.onedata.in_stock_total_sum ='￥' +  (self.onedata.in_stock_total_sum * 0.01).toFixed(2)
          self.onedata.current_stock_total_sum ='￥' +  (self.onedata.current_stock_total_sum * 0.01).toFixed(2)
          self.onedata.out_stock_total_sum ='￥' +  (self.onedata.out_stock_total_sum * 0.01).toFixed(2)
        },function(err){})
      },
//      获取仓库列表
      getProviderList: function(){
        var self = this
        var data = {}
        var url = requestSystemUrl + '/backend-system/warehouse-minimal-list'
        getDataFromApi(url,data,function(response){
          self.providerList = response.data.body.list
        })
      },
//      搜索
      searchMethod: function(page){
        this.getListData(page)
        this.getOneData()
      },
//      撤销搜索
      searchCancel: function(){
        var self = this
        self.searchData.start_time = ''
        self.searchData.end_time = ''
        self.searchData.item_code = ''
        self.searchData.item_name = ''
        self.searchData.warehouse_id = ''
        this.getListData(1)
        this.getOneData()
      },
//    对获取到的数据进行处理1
      modifyGetedData: function (data) {
        function priceDetail(price) {
          if(price != ''){
            price = '￥' + ((price * (0.01)).toFixed(2))
            return price
          }
        }
        $.each(data, function (index, value) {
          value.current_stock_price =  priceDetail(value.current_stock_price)
          value.in_stock_price = priceDetail(value.in_stock_price)
          value.out_stock_price = priceDetail(value.out_stock_price)
          value.start_stock_price = priceDetail(value.start_stock_price)
          value.start_stock_total_sum =  priceDetail(value.start_stock_total_sum)
          value.in_stock_total_sum = priceDetail(value.in_stock_total_sum)
          value.out_stock_total_sum = priceDetail(value.out_stock_total_sum)
          value.current_stock_total_sum = priceDetail(value.current_stock_total_sum)
        })
      }
    },
    computed: {
//      导出 /backend-system/data-center/purchase/order
      exports: function () {
        var url = requestSystemUrl + '/backend-system/' + token + '/export' + '/data-center/stock/log-collect-list'
        var data =
          'item_name=' + this.searchData.item_name + '&' +
          'item_code=' + this.searchData.item_code + '&' +
          'warehouse_id=' + this.searchData.warehouse_id + '&' +
          'start_time=' + this.searchData.start_time + '&' +
          'end_time=' + this.searchData.end_time
        return this.exportUrl = url + '/export-excel?' + data
      }
    },
    data: function () {
      return {
        exportUrl: '',
        page: [],
        listdata: [],
        onedata: [],
        providerList: [],
        gridColumns: {
          warehouse_name: '仓库',
          goods_code: '货号',
          goods_name: '品名',
          unit_name: '单位',
          unit_specification: '单位规格',
          start_stock: '期初库存数量',
          start_stock_price: '期初平均单价',
          start_stock_total_sum: '期初金额',
          in_stock: '期间入库数量',
          in_stock_price: '期间入库平均单价',
          in_stock_total_sum: '期间入库金额',
          out_stock: '期间出库数量',
          out_stock_price: '期间出库平均单价',
          out_stock_total_sum: '期间出库金额',
          current_stock: '期末库存数量',
          current_stock_price: '期末平均单价',
          current_stock_total_sum: '期末金额'
        },
        productOperate: false,
        searchData: {
          start_time: '',
          end_time: '',
          warehouse_id: '',
          item_code: '',
          item_name: ''
        },
        modal: {
          errModal: false,
          errInfo: ''
        }
      }
    }
  }
</script>
