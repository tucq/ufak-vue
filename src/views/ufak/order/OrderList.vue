<template>
  <a-card :bordered="false">

    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">

          <a-col :md="6" :sm="8">
            <a-form-item label="订单状态">
              <a-select
                placeholder="请选择"
                v-model="queryParam.orderStatus"
              >
                <a-select-option :value="option.code" v-for="option in orderStatusList">
                  {{ option.name }}
                </a-select-option>
              </a-select>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="订单编号">
              <a-input placeholder="请输入订单编号" v-model="queryParam.orderNo"></a-input>
            </a-form-item>
          </a-col>
          <template v-if="toggleSearchStatus">
            <a-col :md="6" :sm="8">
              <a-form-item label="收货电话号码">
                <a-input placeholder="请输入收货电话号码" v-model="queryParam.telephone"></a-input>
              </a-form-item>
            </a-col>
          </template>
          <a-col :md="6" :sm="8">
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
      <!--<a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>-->
      <a-button type="primary" icon="download" @click="handleExportXls('订单主表')">导出</a-button>
      <!--<a-upload name="file" :showUploadList="false" :multiple="false" :headers="tokenHeader" :action="importExcelUrl" @change="handleImportExcel">-->
      <!--<a-button type="primary" icon="import">导入</a-button>-->
      <!--</a-upload>-->
      <!--<a-dropdown v-if="selectedRowKeys.length > 0">-->
      <!--<a-menu slot="overlay">-->
      <!--<a-menu-item key="1" @click="batchDel"><a-icon type="delete"/>删除</a-menu-item>-->
      <!--</a-menu>-->
      <!--<a-button style="margin-left: 8px"> 批量操作 <a-icon type="down" /></a-button>-->
      <!--</a-dropdown>-->
    </div>

    <!-- table区域-begin -->
    <div>
      <!--<div class="ant-alert ant-alert-info" style="margin-bottom: 16px;">-->
      <!--<i class="anticon anticon-info-circle ant-alert-icon"></i> 已选择 <a style="font-weight: 600">{{ selectedRowKeys.length }}</a>项-->
      <!--<a style="margin-left: 24px" @click="onClearSelected">清空</a>-->
      <!--</div>-->

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

        <span slot="orderStatus" slot-scope="text, record">
            <a-tag v-if="record.orderStatus === '0'" color="cyan">待付款</a-tag>
            <a-tag v-else-if="record.orderStatus === '1'" color="#f50">待发货</a-tag>
            <a-tag v-else-if="record.orderStatus === '2'" color="blue">待收货</a-tag>
            <a-tag v-else-if="record.orderStatus === '3'" color="pink">已取消</a-tag>
            <a-tag v-else-if="record.orderStatus === '4'" color="green">已完成</a-tag>
            <a-tag v-else-if="record.orderStatus === '5'" color="pink">已退款</a-tag>
            <a-tag v-else-if="record.orderStatus === '6'" color="#cd201f">售后待审核</a-tag>
            <a-tag v-else="record.orderStatus == null || record.orderStatus == ''" color="red">异常</a-tag>
        </span>

        <span slot="action" slot-scope="text, record">
          <a v-if="record.orderStatus === '1'" @click="sendGoods(record)">发货</a>
          <a-divider v-if="record.orderStatus === '1'" type="vertical"/>
          <a @click="handleEdit(record)">查看</a>
          <a-divider type="vertical"/>
          <a @click="changePrice(record)">改价</a>
          <!--<a-dropdown>-->
          <!--<a class="ant-dropdown-link">更多 <a-icon type="down"/></a>-->
          <!--<a-menu slot="overlay">-->
          <!--<a-menu-item @click="handleEdit(record)">查看</a-menu-item>-->
          <!--<a-menu-item>-->
          <!--<a-menu-item @click="changePrice(record)">改价</a-menu-item>-->
          <!--</a-menu-item>-->
          <!--</a-menu>-->
          <!--</a-dropdown>-->
        </span>

      </a-table>
    </div>
    <!-- table区域-end -->

    <!-- 表单区域 -->
    <order-modal ref="modalForm" @ok="modalFormOk"></order-modal>
    <send-goods ref="sendGoods" @ok="modalFormOk"></send-goods>
  </a-card>
</template>

<script>
  import OrderModal from './modules/OrderModal'
  import SendGoods from './modules/SendGoods'
  import {JeecgListMixin} from '@/mixins/JeecgListMixin'

  export default {
    name: "OrderList",
    mixins: [JeecgListMixin],
    components: {
      OrderModal,
      SendGoods
    },
    data() {
      return {
        description: '订单主表管理页面',
        orderStatusList: [
          {code: '', name: '全部'},
          {code: '0', name: '待付款'},
          {code: '1', name: '待发货'},
          {code: '2', name: '待收货'},
          {code: '3', name: '已取消'},
          {code: '4', name: '已完成'},
          {code: '5', name: '已退款'},
          {code: '6', name: '售后待审核'}
        ],
        // 表头
        columns: [
          {
            title: '订单编号',
            align: "center",
            dataIndex: 'orderNo'
          },
          {
            title: '实收金额',
            align: "center",
            dataIndex: 'paymentAmount'
          },
          {
            title: '订单状态',
            align: "center",
            dataIndex: 'orderStatus',
            scopedSlots: {customRender: 'orderStatus'},
          },
          {
            title: '收货人',
            align: "center",
            dataIndex: 'usernameTel'
          },
          {
            title: '收货地址',
            align: "center",
            dataIndex: 'userAddress'
          },
          {
            title: '创建时间',
            align: "center",
            dataIndex: 'createTime'
          },
          {
            title: '操作',
            dataIndex: 'action',
            align: "left",
            scopedSlots: {customRender: 'action'},
          }
        ],
        url: {
          list: "/order/list",
          exportXlsUrl: "/order/exportXls",
          importExcelUrl: "/order/importExcel",
        },
      }
    },
    computed: {
      importExcelUrl: function () {
        return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`;
      }
    },
    methods: {
      sendGoods(record) {
        this.$refs.sendGoods.edit(record);
        this.$refs.sendGoods.title = "发货";
        this.$refs.sendGoods.disableSubmit = true;
      },
      changePrice(record) {
        alert("开发中..." + record.id);
      },
    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less'
</style>