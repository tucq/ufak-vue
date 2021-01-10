<template>
  <a-card :bordered="false">

    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <a-col :md="6" :sm="8">
            <a-form-item label="分类">
              <a-select
                placeholder="请输入分类"
                v-model="queryParam.categoryId"
                :defaultValue="{ key: '' }"
              >
                <a-select-option :value="option.id" v-for="option in categoryList">
                  {{ option.itemText }}
                </a-select-option>
              </a-select>
            </a-form-item>

          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="标题">
              <a-input placeholder="请输入标题" v-model="queryParam.title"></a-input>
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
          <span slot="state" slot-scope="text, record">
              <a-tag v-if="record.state === '0'" color="green">显示</a-tag>
              <a-tag v-else-if="record.state === '1'" color="red">禁言</a-tag>
          </span>
          <span slot="action" slot-scope="text, record">
          <a @click="handleEdit(record)">编辑</a>
          <a-divider type="vertical" />
           <a @click="handleTop(record)">置顶</a>
           <a-divider type="vertical" />
            <a-dropdown>
            <a class="ant-dropdown-link">更多 <a-icon type="down" /></a>
            <a-menu slot="overlay">
              <a-menu-item>
                <a-popconfirm :title="record.state === '0'?'确定禁言吗?':'确定取消禁言吗?'" @confirm="() => forbidden(record)">
                  <a v-if="record.state === '0'">禁言</a>
                  <a v-if="record.state === '1'">解禁</a>
                </a-popconfirm>
              </a-menu-item>
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
    <postBar-modal ref="modalForm" @ok="modalFormOk"></postBar-modal>
    <top-bar-modal ref="topBarForm" @ok="modalFormOk"></top-bar-modal>
  </a-card>
</template>

<script>
  import PostBarModal from './modules/PostBarModal'
  import TopBarModal from './modules/TopBarModal'
  import { getAction,postAction } from '@/api/manage'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'

  export default {
    name: "PostBarList",
    mixins:[JeecgListMixin],
    components: {
      PostBarModal,
      TopBarModal
    },
    data () {
      return {
        description: '贴吧管理页面',
        categoryList:[],
        // 表头
        columns: [
		   {
            title: '分类',
            width:'10%',
            align:"center",
            dataIndex: 'categoryText'
           },
		   {
            title: '标题',
            width:'35%',
            align:"center",
            dataIndex: 'title'
           },
		   {
            title: '用户昵称',
            width:'13%',
            align:"center",
            dataIndex: 'realname'
           },
          {
            title: '观注数量',
            width:'5%',
            align:"center",
            dataIndex: 'fansNum'
          },
          {
            title: '回复数量',
            width:'5%',
            align:"center",
            dataIndex: 'replyNum'
          },
		   {
            title: '分享数量',
            width:'5%',
            align:"center",
            dataIndex: 'shareNum'
           },
		   {
            title: '点赞数量',
            width:'5%',
            align:"center",
            dataIndex: 'likesNum'
           },
		   {
            title: '状态',
            width:'5%',
            align:"center",
            dataIndex: 'stateText',
            scopedSlots: { customRender: 'state' },
           },
		   {
            title: '置顶顺序',
            width:'5%',
            align:"center",
            dataIndex: 'top'
           },
          {
            title: '操作',
            width:'12%',
            dataIndex: 'action',
            align:"center",
            scopedSlots: { customRender: 'action' },
          }
        ],
		url: {
          list: "/postBar/list",
          delete: "/postBar/delete",
          forbidden: "/postBar/forbidden",
          categoryList: "/sys/dict/item/bar_category",
       },
    }
  },
  computed: {

  },
    created(){
      this.getCategoryList();
    },
    methods: {
      getCategoryList(){
        getAction(this.url.categoryList).then((res)=>{
          if(res.success){
            this.categoryList = res.result;
          }
        });
      },
      handleTop(record){
        this.$refs.topBarForm.edit(record);
        this.$refs.topBarForm.title = "置顶";
      },
      forbidden(record){
        postAction(this.url.forbidden,record).then((res)=>{
          if(res.success){
            this.$message.success(res.message);
            this.loadData();
          }
        });
      },
//      lineChange(value){
//        this.queryParam.lineId = value;
//      },
    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less'
</style>