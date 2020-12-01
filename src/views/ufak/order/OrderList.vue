<template>
  <a-card :bordered="false">

    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">

          <a-col :md="6" :sm="8">
            <a-form-item label="用户id">
              <a-input placeholder="请输入用户id" v-model="queryParam.userId"></a-input>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="订单编号">
              <a-input placeholder="请输入订单编号" v-model="queryParam.orderNo"></a-input>
            </a-form-item>
          </a-col>
        <template v-if="toggleSearchStatus">
        <a-col :md="6" :sm="8">
            <a-form-item label="物流编号">
              <a-input placeholder="请输入物流编号" v-model="queryParam.logisticsNo"></a-input>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="订单金额">
              <a-input placeholder="请输入订单金额" v-model="queryParam.orderAmount"></a-input>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="活动优惠券">
              <a-input placeholder="请输入活动优惠券" v-model="queryParam.eventAmount"></a-input>
            </a-form-item>
          </a-col>
          </template>
          <a-col :md="6" :sm="8" >
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
              <a @click="handleToggleSearch" style="margin-left: 8px">
                {{ toggleSearchStatus ? '收起' : '展开' }}
                <a-icon :type="toggleSearchStatus ? 'up' : 'down'"/>
              </a>
            </span>
          </a-col>

        </a-row>
      </a-form>
    </div>

    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>
      <a-button type="primary" icon="download" @click="handleExportXls('订单主表')">导出</a-button>
      <a-upload name="file" :showUploadList="false" :multiple="false" :headers="tokenHeader" :action="importExcelUrl" @change="handleImportExcel">
        <a-button type="primary" icon="import">导入</a-button>
      </a-upload>
      <a-dropdown v-if="selectedRowKeys.length > 0">
        <a-menu slot="overlay">
          <a-menu-item key="1" @click="batchDel"><a-icon type="delete"/>删除</a-menu-item>
        </a-menu>
        <a-button style="margin-left: 8px"> 批量操作 <a-icon type="down" /></a-button>
      </a-dropdown>
    </div>

    <!-- table区域-begin -->
    <div>
      <div class="ant-alert ant-alert-info" style="margin-bottom: 16px;">
        <i class="anticon anticon-info-circle ant-alert-icon"></i> 已选择 <a style="font-weight: 600">{{ selectedRowKeys.length }}</a>项
        <a style="margin-left: 24px" @click="onClearSelected">清空</a>
      </div>

      <a-table
        ref="table"
        size="middle"
        bordered
        rowKey="id"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"
        @change="handleTableChange">

        <span slot="action" slot-scope="text, record">
          <a @click="handleEdit(record)">编辑</a>

          <a-divider type="vertical" />
          <a-dropdown>
            <a class="ant-dropdown-link">更多 <a-icon type="down" /></a>
            <a-menu slot="overlay">
              <a-menu-item>
                <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.id)">
                  <a>删除</a>
                </a-popconfirm>
              </a-menu-item>
            </a-menu>
          </a-dropdown>
        </span>

      </a-table>
    </div>
    <!-- table区域-end -->

    <!-- 表单区域 -->
    <order-modal ref="modalForm" @ok="modalFormOk"></order-modal>
  </a-card>
</template>

<script>
  import OrderModal from './modules/OrderModal'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'

  export default {
    name: "OrderList",
    mixins:[JeecgListMixin],
    components: {
      OrderModal
    },
    data () {
      return {
        description: '订单主表管理页面',
        // 表头
        columns: [
          {
            title: '#',
            dataIndex: '',
            key:'rowIndex',
            width:60,
            align:"center",
            customRender:function (t,r,index) {
              return parseInt(index)+1;
            }
           },
		   {
            title: '用户id',
            align:"center",
            dataIndex: 'userId'
           },
		   {
            title: '订单编号',
            align:"center",
            dataIndex: 'orderNo'
           },
		   {
            title: '物流编号',
            align:"center",
            dataIndex: 'logisticsNo'
           },
		   {
            title: '订单金额',
            align:"center",
            dataIndex: 'orderAmount'
           },
		   {
            title: '活动优惠券',
            align:"center",
            dataIndex: 'eventAmount'
           },
		   {
            title: '优惠券',
            align:"center",
            dataIndex: 'couponAmount'
           },
		   {
            title: '订单状态: 0-待付款,1-待发货,2-待收货,3-已取消,4-已完成',
            align:"center",
            dataIndex: 'orderStatus'
           },
		   {
            title: '收货姓名',
            align:"center",
            dataIndex: 'username'
           },
		   {
            title: '收货电话号码',
            align:"center",
            dataIndex: 'telephone'
           },
		   {
            title: '地区',
            align:"center",
            dataIndex: 'address'
           },
		   {
            title: '详细地址',
            align:"center",
            dataIndex: 'detailAddress'
           },
		   {
            title: '微信支付订单号',
            align:"center",
            dataIndex: 'transactionId'
           },
		   {
            title: '微信支付回调签名',
            align:"center",
            dataIndex: 'sign'
           },
		   {
            title: '微信订单金额(分)',
            align:"center",
            dataIndex: 'totalFee'
           },
		   {
            title: '微信现金支付金额(分)',
            align:"center",
            dataIndex: 'cashFee'
           },
		   {
            title: '微信支付回调业务结果',
            align:"center",
            dataIndex: 'resultCode'
           },
          {
            title: '操作',
            dataIndex: 'action',
            align:"center",
            scopedSlots: { customRender: 'action' },
          }
        ],
		url: {
          list: "/com.ufak/order/list",
          delete: "/com.ufak/order/delete",
          deleteBatch: "/com.ufak/order/deleteBatch",
          exportXlsUrl: "com.ufak/order/exportXls",
          importExcelUrl: "com.ufak/order/importExcel",
       },
    }
  },
  computed: {
    importExcelUrl: function(){
      return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`;
    }
  },
    methods: {
     
    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less'
</style>