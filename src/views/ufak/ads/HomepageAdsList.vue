<template>
  <a-row :gutter="10">
    <a-col :md="12" :sm="24">
      <a-card :bordered="false">
        <div class="table-operator">
          <a-button @click="handleAdd" type="primary" icon="plus">新增广告</a-button>
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
            <span slot="imgUrl" slot-scope="imgUrl">
                <a-row>
                  <img v-if="imgUrl != ''" :src="getAvatarView(imgUrl)" style="height:50px;max-width:50px"/>
                </a-row>
            </span>
            <span slot="action" slot-scope="text, record">
          <a @click="handleEdit(record)">编辑</a>
          <a-divider type="vertical"/>
          <a @click="handleEdit(record)">设置商品</a>
          <a-divider type="vertical"/>
          <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.id)">
            <a>删除</a>
          </a-popconfirm>
        </span>

          </a-table>
        </div>

        <homepageAds-modal ref="modalForm" @ok="modalFormOk"></homepageAds-modal>
      </a-card>
    </a-col>

    <a-col :md="12" :sm="24">
      <a-card :bordered="false">
        <div class="table-operator">
          <a-button @click="addProduct" type="primary" icon="plus">新增商品</a-button>
        </div>

        <!-- table区域-begin -->
        <div>
          <a-table
            ref="table"
            size="middle"
            bordered
            rowKey="id"
            :columns="productColumns"
            :dataSource="productDataSource"
            :pagination="ipagination"
            :loading="loading"
            @change="handleTableChange">
            <span slot="imgUrl" slot-scope="imgUrl">
                <a-row>
                  <img v-if="imgUrl != ''" :src="getAvatarView(imgUrl)" style="height:50px;max-width:50px"/>
                </a-row>
            </span>
            <span slot="action" slot-scope="text, record">
              <a @click="handleEdit(record)">浏览</a>
              <a-divider type="vertical"/>
              <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.id)">
                <a>删除</a>
              </a-popconfirm>
            </span>
          </a-table>
        </div>

        <ads-product-modal ref="adsForm" ></ads-product-modal>
      </a-card>
    </a-col>
  </a-row>


</template>

<script>
  import HomepageAdsModal from './modules/HomepageAdsModal'
  import AdsProductModal from './modules/AdsProductModal'
  import {JeecgListMixin} from '@/mixins/JeecgListMixin'
//  import ARow from "ant-design-vue/es/grid/Row";

  export default {
    name: "HomepageAdsList",
    mixins: [JeecgListMixin],
    components: {
//      ARow,
      HomepageAdsModal,
      AdsProductModal
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
            title: '广告图片',
            width: '20%',
            align: "center",
            dataIndex: 'imgUrl',
            scopedSlots: {customRender: 'imgUrl'},
          },
          {
            title: '广告名称',
            width: '25%',
            align: "center",
            dataIndex: 'adsName'
          },
          {
            title: '排版',
            width: '10%',
            align: "center",
            dataIndex: 'layout_dictText'
          },
          {
            title: '状态',
            width: '15%',
            align: "center",
            dataIndex: 'state_dictText'
          },
          {
            title: '操作',
            width: '30%',
            dataIndex: 'action',
            align: "center",
            scopedSlots: {customRender: 'action'},
          }
        ],
        productDataSource:[],
        productColumns: [
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
            title: '商品图片',
            width: '20%',
            align: "center",
            dataIndex: 'imgUrl',
            scopedSlots: {customRender: 'imgUrl'},
          },
          {
            title: '商品名称',
            width: '25%',
            align: "center",
            dataIndex: 'adsName'
          },
          {
            title: '状态',
            width: '15%',
            align: "center",
            dataIndex: 'state_dictText'
          },
          {
            title: '操作',
            width: '30%',
            dataIndex: 'action',
            align: "center",
            scopedSlots: {customRender: 'action'},
          }
        ],
        url: {
          list: "/homepageAds/list",
          delete: "/homepageAds/delete",
          deleteBatch: "/homepageAds/deleteBatch",
          exportXlsUrl: "/homepageAds/exportXls",
          importExcelUrl: "/homepageAds/importExcel",
          imgerver: window._CONFIG['domianURL'] + "/sys/common/view",
        },
      }
    },
    computed: {
      importExcelUrl: function () {
        return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`;
      }
    },
    methods: {
      getAvatarView(image){
        return this.url.imgerver + "/" + image;
      },
      addProduct(){
          this.$refs.adsForm.title = "新增";
          this.$refs.adsForm.visible = true;
      }
    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less'
</style>