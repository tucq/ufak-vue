<template>
  <a-card :bordered="false">

    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <a-col :md="6" :sm="8">
            <a-form-item label="状态">
              <a-select
                placeholder="请输状态"
                v-model="queryParam.state"
              >
                <a-select-option value="">全部</a-select-option>
                <a-select-option value="0">启用</a-select-option>
                <a-select-option value="1">停用</a-select-option>
              </a-select>
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
      <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>
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
        <span slot="specsImage" slot-scope="specsImage">
            <a-row>
              <img :src="getAvatarView(specsImage)" style="height:50px;max-width:50px"/>
            </a-row>
        </span>
        <span slot="state" slot-scope="text, record">
            <a-tag v-if="record.state === '0'" color="green">启用</a-tag>
            <a-tag v-else-if="record.state === '1'" color="red">停用</a-tag>
        </span>
        <span slot="action" slot-scope="text, record">
          <a @click="handleEdit(record)">编辑</a>
          <a-divider type="vertical" />
          <a v-if="record.state === '0'" @click="stop(record.id)">禁用</a>
          <a v-if="record.state === '1'" @click="start(record.id)">启用</a>
          <a-divider type="vertical" />
          <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.id)">
            <a>删除</a>
          </a-popconfirm>
        </span>

      </a-table>
    </div>
    <!-- table区域-end -->

    <!-- 表单区域 -->
    <flashSale-modal ref="modalForm" @ok="modalFormOk"></flashSale-modal>
  </a-card>
</template>

<script>
  import FlashSaleModal from './modules/FlashSaleModal'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import { getAction,postAction } from '@/api/manage'

  export default {
    name: "FlashSaleList",
    mixins:[JeecgListMixin],
    components: {
      FlashSaleModal
    },
    data () {
      return {
        description: '限制抢购管理页面',
        // 表头
        columns: [
		   {
            title: '商品图片',
            align:"center",
            width:"10%",
            dataIndex: 'specsImage',
            scopedSlots: {customRender: 'specsImage'},
           },
          {
            title: '商品名称',
            align:"center",
            width:"15%",
            dataIndex: 'productName'
          },
		   {
            title: '标题',
            align:"center",
            width:"10%",
            dataIndex: 'title'
           },
		   {
            title: '描述',
            align:"center",
            width:"10%",
            dataIndex: 'remark'
           },
		   {
            title: '秒杀价',
            align:"center",
            width:"7%",
            dataIndex: 'killPrice'
           },
		   {
            title: '抢购数量',
            align:"center",
            width:"7%",
            dataIndex: 'stock'
           },
		   {
            title: '状态',
            align:"center",
            width:"5%",
            dataIndex: 'state',
            scopedSlots: {customRender: 'state'},
           },
		   {
            title: '开始时间',
            align:"center",
            width:"13%",
            dataIndex: 'startTime'
           },
		   {
            title: '结束时间',
            align:"center",
            width:"13%",
            dataIndex: 'endTime'
           },
          {
            title: '操作',
            dataIndex: 'action',
            width:"10%",
            align:"center",
            scopedSlots: { customRender: 'action' },
          }
        ],
		url: {
          list: "/flashSale/list",
          delete: "/flashSale/delete",
          stop: "/flashSale/stop",
          start: "/flashSale/start",
          imgerver: window._CONFIG['domianURL']+"/sys/common/view",
       },
    }
  },
  computed: {

  },
    methods: {
      getAvatarView(image){
        return this.url.imgerver +"/"+ image;
      },
      stop(id){
        postAction(this.url.stop+"?id="+id,null).then((res)=>{
          if(res.success){
            this.$message.success(res.message);
            this.loadData();
          }
        });
      },
      start(id){
        postAction(this.url.start+"?id="+id,null).then((res)=>{
          if(res.success){
            this.$message.success(res.message);
            this.loadData();
          }
        });
      }
    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less'
</style>