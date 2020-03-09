<template>
  <a-card :bordered="false">

    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <a-col :md="6" :sm="8">
            <a-form-item label="商品名称">
              <a-input placeholder="请输入商品名称" v-model="queryParam.type"></a-input>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
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
      <a-dropdown v-if="selectedRowKeys.length > 0">
        <a-menu slot="overlay">
          <a-menu-item key="1" @click="batchDel">
            <a-icon type="delete"/>
            删除
          </a-menu-item>
        </a-menu>
      </a-dropdown>
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

        <span slot="action" slot-scope="text, record">
          <a @click="handleEdit(record)">编辑</a>
          <a-divider type="vertical"/>
          <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.id)">
            <a>删除</a>
          </a-popconfirm>
        </span>

      </a-table>
    </div>
    <!-- table区域-end -->

    <!-- 表单区域 -->
    <homepageAds-modal ref="modalForm" @ok="modalFormOk"></homepageAds-modal>
  </a-card>
</template>

<script>
  import HomepageAdsModal from './modules/HomepageAdsModal'
  import {JeecgListMixin} from '@/mixins/JeecgListMixin'

  export default {
    name: "HomepageAdsList",
    mixins: [JeecgListMixin],
    components: {
      HomepageAdsModal
    },
    data () {
      return {
        description: '首页广告管理页面',
        // 表头
        columns: [
          {
            title: '#',
            dataIndex: '',
            key: 'rowIndex',
            width: 60,
            align: "center",
            customRender: function (t, r, index) {
              return parseInt(index) + 1;
            }
          },
          {
            title: '图片',
            align: "center",
            dataIndex: 'imgUrl'
          },
          {
            title: '类型',
            align: "center",
            dataIndex: 'type'
          },
          {
            title: '头部广告',
            align: "center",
            dataIndex: 'headImg'
          },
          {
            title: '排版',
            align: "center",
            dataIndex: 'layout'
          },
          {
            title: '背景色',
            align: "center",
            dataIndex: 'bgColor'
          },
          {
            title: '排序',
            align: "center",
            dataIndex: 'sort'
          },
          {
            title: '状态',
            align: "center",
            dataIndex: 'state'
          },
          {
            title: '操作',
            dataIndex: 'action',
            align: "center",
            scopedSlots: {customRender: 'action'},
          }
        ],
        url: {
          list: "/com.ufak/homepageAds/list",
          delete: "/com.ufak/homepageAds/delete",
          deleteBatch: "/com.ufak/homepageAds/deleteBatch",
          exportXlsUrl: "com.ufak/homepageAds/exportXls",
          importExcelUrl: "com.ufak/homepageAds/importExcel",
        },
      }
    },
    computed: {
      importExcelUrl: function () {
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