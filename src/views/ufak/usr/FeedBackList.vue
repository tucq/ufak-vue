<template>
  <a-card :bordered="false">

    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">

          <a-col :md="6" :sm="8">
            <a-form-item label="内容">
              <a-input placeholder="请输入内容" v-model="queryParam.content"></a-input>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="图片路径，多个逗号隔开">
              <a-input placeholder="请输入图片路径，多个逗号隔开" v-model="queryParam.images"></a-input>
            </a-form-item>
          </a-col>
        <template v-if="toggleSearchStatus">
        <a-col :md="6" :sm="8">
            <a-form-item label="反馈人">
              <a-input placeholder="请输入反馈人" v-model="queryParam.username"></a-input>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="联系电话">
              <a-input placeholder="请输入联系电话" v-model="queryParam.telephone"></a-input>
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
        <span slot="images" slot-scope="images">
            <a-row>
              <img :src="getAvatarView(images.split(',')[0])" style="height:50px;max-width:50px"/>
            </a-row>
        </span>
        <span slot="action" slot-scope="text, record">
          <a @click="handleEdit(record)">编辑</a>
          <a-divider type="vertical" />
          <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.id)">
            <a>删除</a>
          </a-popconfirm>
        </span>

      </a-table>
    </div>
    <!-- table区域-end -->

    <!-- 表单区域 -->
    <feedBack-modal ref="modalForm" @ok="modalFormOk"></feedBack-modal>
  </a-card>
</template>

<script>
  import FeedBackModal from './modules/FeedBackModal'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'

  export default {
    name: "FeedBackList",
    mixins:[JeecgListMixin],
    components: {
      FeedBackModal
    },
    data () {
      return {
        description: '意见反馈管理页面',
        // 表头
        columns: [
          {
            title: '#',
            dataIndex: '',
            key:'rowIndex',
            width: '4%',
            align:"center",
            customRender:function (t,r,index) {
              return parseInt(index)+1;
            }
           },
		   {
            title: '内容',
            align:"left",
            dataIndex: 'content',
            width: '51%',
           },
		   {
            title: '图片',
            align:"center",
             width: '10%',
             dataIndex: 'images',
            scopedSlots: {customRender: 'images'},
           },
		   {
            title: '反馈人',
            align:"center",
         width: '10%',
            dataIndex: 'username'
           },
		   {
            title: '联系电话',
            align:"center",
         width: '10%',
            dataIndex: 'telephone'
           },
          {
            title: '操作',
            dataIndex: 'action',
            align:"center",
            width: '10%',
            scopedSlots: { customRender: 'action' },
          }
        ],
		url: {
          list: "/feedBack/list",
          delete: "/feedBack/delete",
          deleteBatch: "/feedBack/deleteBatch",
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
    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less'
</style>