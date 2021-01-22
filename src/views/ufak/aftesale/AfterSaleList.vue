<template>
  <a-card :bordered="false">

    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <a-col :md="6" :sm="8">
            <a-form-item label="状态">
              <a-select
                placeholder="请选择"
                v-model="queryParam.status"
              >
                <a-select-option value="">全部</a-select-option>
                <a-select-option value="0">待处理</a-select-option>
                <a-select-option value="1">已完成</a-select-option>
                <a-select-option value="2">客户取消</a-select-option>
              </a-select>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="退款售后单号">
              <a-input placeholder="请输入退款售后单号" v-model="queryParam.afterSaleNo"></a-input>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8" >
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
            </span>
          </a-col>
        </a-row>
      </a-form>
    </div>

    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <a-button type="primary" icon="download" @click="handleExportXls('售后处理')">导出</a-button>
    </div>

    <!-- table区域-begin -->
    <div>
      <a-table
        ref="table"
        size="middle"
        bordered
        rowKey="id"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        @change="handleTableChange">
        <span slot="serviceType" slot-scope="text, record">
            <a-tag v-if="record.serviceType === '0'" color="red">退款</a-tag>
            <a-tag v-else-if="record.serviceType === '1'" color="orange">退货</a-tag>
            <a-tag v-else-if="record.serviceType === '2'" color="">换货</a-tag>
            <a-tag v-else-if="record.serviceType === '3'" color="">维修</a-tag>
            <a-tag v-else-if="record.serviceType === '4'" color="">开票</a-tag>
            <a-tag v-else="record.status == null || record.status == ''" color="red">异常</a-tag>
        </span>
        <span slot="status" slot-scope="text, record">
            <a-tag v-if="record.status === '0'" color="#f50">待处理</a-tag>
            <a-tag v-else-if="record.status === '1'" color="#87d068">已完成</a-tag>
            <a-tag v-else-if="record.status === '2'" color="#2db7f5">客户取消</a-tag>
            <a-tag v-else="record.status == null || record.status == ''" color="red">异常</a-tag>
        </span>
        <span slot="action" slot-scope="text, record">
          <a v-if="record.status === '0'" @click="handleEdit(record)">处理</a>
          <a v-if="record.status != '0'" @click="handleView(record)">查看</a>
        </span>
      </a-table>
    </div>
    <!-- table区域-end -->

    <!-- 表单区域 -->
    <afterSale-modal ref="modalForm" @ok="modalFormOk"></afterSale-modal>
    <invoice-modal ref="invoiceForm" @ok="modalFormOk"></invoice-modal>
  </a-card>
</template>

<script>
  import AfterSaleModal from './modules/AfterSaleModal'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import InvoiceModal from "./modules/InvoiceModal";

  export default {
    name: "AfterSaleList",
    mixins:[JeecgListMixin],
    components: {
      InvoiceModal, AfterSaleModal
    },
    data () {
      return {
        description: '售后处理管理页面',
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
            title: '售后单号',
            align:"center",
            dataIndex: 'afterSaleNo'
           },
		   {
            title: '售后类型',
            align:"center",
            dataIndex: 'serviceType',
            scopedSlots: { customRender: 'serviceType' },
           },
		   {
            title: '状态',
            align:"center",
            dataIndex: 'status',
            scopedSlots: { customRender: 'status' },
           },
		   {
            title: '微信支付订单号',
            align:"center",
            dataIndex: 'transactionId'
           },
          {
            title: '创建时间',
            align:"center",
            dataIndex: 'createTime'
          },
          {
            title: '操作',
            dataIndex: 'action',
            align:"center",
            scopedSlots: { customRender: 'action' },
          }
        ],
		url: {
          list: "/afterSale/list",
          delete: "/afterSale/delete",
          deleteBatch: "/afterSale/deleteBatch",
          exportXlsUrl: "/afterSale/exportXls",
       },
    }
  },
  computed: {
    importExcelUrl: function(){
      return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`;
    }
  },
    methods: {
      handleEdit: function (record) {
        if(record.serviceType == "0"){
          this.$refs.modalForm.title = "退款处理";
          this.$refs.modalForm.loadData(record);
          this.$refs.modalForm.disableSubmit = false;
        }else if(record.serviceType == "1"){
          this.$refs.modalForm.title = "退货处理";
        }else if(record.serviceType == "2"){
          this.$refs.modalForm.title = "换货处理";
        }else if(record.serviceType == "3"){
          this.$refs.modalForm.title = "维修处理";
        }else if(record.serviceType == "4"){
          this.$refs.invoiceForm.title = "开票处理";
          this.$refs.invoiceForm.loadData(record);
          this.$refs.invoiceForm.disableSubmit = false;
        }

      },
      handleView: function (record) {
        if(record.serviceType == "0"){
          this.$refs.modalForm.loadData(record);
          this.$refs.modalForm.title = "退款查看";
          this.$refs.modalForm.disableSubmit = true;
        }else if(record.serviceType == "1"){
          this.$refs.modalForm.title = "退货处理";
        }else if(record.serviceType == "2"){
          this.$refs.modalForm.title = "换货处理";
        }else if(record.serviceType == "3"){
          this.$refs.modalForm.title = "维修处理";
        }else if(record.serviceType == "4"){
          this.$refs.invoiceForm.loadData(record);
          this.$refs.invoiceForm.title = "开票查看";
          this.$refs.invoiceForm.disableSubmit = true;

        }

      },
    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less'
</style>