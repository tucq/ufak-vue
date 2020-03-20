<template>
  <a-row :gutter="10">
    <a-col :md="11" :sm="24">
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
            @rowClick="handelSelectRow">
            <span slot="imgUrl" slot-scope="imgUrl">
                <a-row>
                  <img v-if="imgUrl != ''" :src="getAvatarView(imgUrl)" style="height:50px;max-width:50px"/>
                </a-row>
            </span>
            <span slot="action" slot-scope="text, record">
          <a @click="handleEdit(record)">编辑</a>
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

    <a-col :md="13" :sm="24">
      <a-card :bordered="false">
        <div class="table-operator">
          <a-button @click="addProduct" type="primary" icon="plus" :disabled="this.adsId == ''" >新增商品</a-button>
          <span style="padding-left: 20px;">
            广告名称：<span style="color: red;">{{adsName}}</span>
          </span>
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
            :pagination="adsProductPagination"
            :loading="loading"
          >
            <span slot="image" slot-scope="image">
                <a-row>
                  <img v-if="image != ''" :src="getAvatarView(image.split(',')[0])" style="height:50px;max-width:50px"/>
                </a-row>
            </span>
            <span slot="layout" slot-scope="text, record, index">
                <a-row>
                  <a-select @change="e => layoutChange(e, record)" v-model="record.layout" style="width: 80px;">
                    <a-select-option value="0" >单排</a-select-option>
                    <a-select-option value="1" >双排</a-select-option>
                    <a-select-option value="2" >三排</a-select-option>
                  </a-select>
                </a-row>
            </span>
            <span slot="adsAction" slot-scope="text, record">
              <a @click="handleView(record)">浏览</a>
              <a-divider type="vertical"/>
              <a-popconfirm title="确定删除吗?" @confirm="() => handleDeleteAds(record.id)">
                <a>删除</a>
              </a-popconfirm>
            </span>
          </a-table>
        </div>

        <ads-product-modal ref="adsForm" :adsId="adsId" @ok="handelAdsModal"></ads-product-modal>
      </a-card>
    </a-col>
  </a-row>


</template>

<script>
  import HomepageAdsModal from './modules/HomepageAdsModal'
  import AdsProductModal from './modules/AdsProductModal'
  import {JeecgListMixin} from '@/mixins/JeecgListMixin'
  import { getAction,deleteAction } from '@/api/manage'
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
        adsId: "",
        adsName: "",
        btnDisabled: this.adsId == "" ? true:false,
        // 表头
        columns: [
          {
            title: '#',
            dataIndex: '',
            key: 'rowIndex',
            width: '5%',
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
            width: '35%',
            align: "center",
            dataIndex: 'adsName'
          },
          {
            title: '类别',
            width: '20%',
            align: "center",
            dataIndex: 'type_dictText'
          },
          {
            title: '操作',
            width: '20%',
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
            width: '5%',
            align: "center",
            customRender: function (t, r, index) {
              return parseInt(index) + 1;
            }
          },
          {
            title: '商品图片',
            width: '20%',
            align: "center",
            dataIndex: 'image',
            scopedSlots: {customRender: 'image'},
          },
          {
            title: '商品名称',
            width: '30%',
            align: "center",
            dataIndex: 'productName'
          },
          {
              title: '排版',
              width: '18%',
              align: "center",
              dataIndex: 'layout',
              scopedSlots: {customRender: 'layout'},
          },
          {
            title: '销量',
            width: '10%',
            align: "center",
            dataIndex: 'salesVolume'
          },
          {
            title: '操作',
            width: '17%',
            dataIndex: 'action',
            align: "center",
            scopedSlots: {customRender: 'adsAction'},
          }
        ],
        adsProductPagination:{
          current: 1,
          pageSize: 10,
          pageSizeOptions: ['10', '20', '30'],
          showTotal: (total, range) => {
            return range[0] + "-" + range[1] + " 共" + total + "条"
          },
          showQuickJumper: true,
          showSizeChanger: true,
          total: 0
        },
        url: {
          list: "/homepageAds/list",
          delete: "/homepageAds/delete",
          deleteBatch: "/homepageAds/deleteBatch",
          exportXlsUrl: "/homepageAds/exportXls",
          importExcelUrl: "/homepageAds/importExcel",
          imgerver: window._CONFIG['domianURL'] + "/sys/common/view",
          adsList:"/adsProduct/list",
          deleteAdsProduct:"/adsProduct/delete",
        },
      }
    },
    created() {
      this.loadAdsProduct(1);
    },
    methods: {
      loadAdsProduct(arg){
        if(arg===1){
          this.adsProductPagination.current = 1;
        }
        getAction(this.url.adsList,{adsId: this.adsId}).then((res)=>{
          if(res.success){
            this.productDataSource = res.result.records;
            this.adsProductPagination.total = res.result.total;
          }
        })
      },
      getAvatarView(image){
        return this.url.imgerver + "/" + image;
      },
      addProduct(){
          this.$refs.adsForm.title = "新增";
          this.$refs.adsForm.visible = true;
          this.$refs.adsForm.adsId = this.adsId;
          this.$refs.adsForm.selectedRowKeys = [];
      },
      handelSelectRow(record){
          this.adsId = record.id;
          this.adsName = record.adsName;
          this.loadAdsProduct(1);
      },
      handelAdsModal(){
        this.loadAdsProduct();
      },
      handleView(record){
          alert('开发中...');
      },
      handleDeleteAds(id){
          deleteAction(this.url.deleteAdsProduct, {id: id}).then((res) => {
            if (res.success) {
              this.$message.success(res.message);
              this.loadAdsProduct();
            } else {
              this.$message.warning(res.message);
            }
          });
      },
      layoutChange(e,record){
         console.log(e);
         console.log(record);
      }

    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less'
</style>