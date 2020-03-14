<template>
  <a-modal
    :title="title"
    :width="900"
    :visible="visible"
    :confirmLoading="confirmLoading"
    :bodyStyle="{'padding-top': '0px'}"
    @ok="handleOk"
    @cancel="handleCancel"
    cancelText="关闭">
    <a-spin :spinning="confirmLoading">
      <a-layout>
        <a-layout-sider style="background:#ffffff;">
          <product-category-tree ref="categoryTree" @onSelect="selectedCategory"/>
        </a-layout-sider>
        <a-layout-content style="background:#ffffff;">
          <div class="table-page-search-wrapper" style="padding-top:20px;">
            <a-form layout="inline">
              <a-row :gutter="24">
                <a-col :span="14">
                  <a-form-item label="商品名称">
                    <a-input v-model="queryParam.name" placeholder="请输入商品名称" @pressEnter="searchQuery"/>
                  </a-form-item>
                </a-col>
                <a-col :span="8">
                  <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
                    <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
                    <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
                  </span>
                </a-col>
              </a-row>
            </a-form>
          </div>

          <a-table
            size="small"
            bordered
            rowKey="id"
            :columns="columns"
            :dataSource="dataSource"
            :pagination="ipagination"
            >
          </a-table>
        </a-layout-content>
      </a-layout>
    </a-spin>
  </a-modal>
</template>

<script>
  import { postAction,getAction} from '@/api/manage'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import ProductCategoryTree from "@/views/ufak/common/ProductCategoryTree"

  export default {
    name: "HomepageAdsModal",
    mixins:[JeecgListMixin],
    components: {
      ProductCategoryTree,
    },
    data () {
      return {
        title:"操作",
        visible: false,
        queryParam:{},
        dataSource:[],
        confirmLoading: false,
        columns: [
          {
            title: '商品图片',
            width: '25%',
            align: "center",
            dataIndex: 'imgUrl',
            scopedSlots: {customRender: 'image'},
          },
          {
            title: '商品分类',
            width: '25%',
            align: "center",
            dataIndex: 'productTypeName'
          },
          {
            title: '商品名称',
            width: '30%',
            align: "center",
            dataIndex: 'name'
          },
          {
            title: '销量',
            width: '20%',
            align: "center",
            dataIndex: 'salesVolume'
          },

        ],
        ipagination: {
          current: 1,
          pageSize: 10,
          pageSizeOptions: ['10', '20', '30'],
          showTotal: (total, range) => {
            return range[0] + '-' + range[1] + ' 共' + total + '条'
          },
          showQuickJumper: true,
          showSizeChanger: true,
          total: 0
        },
        url: {
          list: "/product/info/list",
        },
      }
    },
    created(){
      this.loadData(1);
    },
    methods: {
      loadData (arg){
        if(arg===1){
          this.ipagination.current = 1;
        }
        let params = this.getQueryParams();//查询条件
        getAction(this.url.list,params).then((res)=>{
          if(res.success){
            this.dataSource = res.result.records;
            this.ipagination.total = res.result.total;
          }
        })
      },
      close () {
        this.$emit('close');
        this.visible = false;
      },
      handleCancel () {
        this.close();
      },
      handleOk () {
        const that = this;
        // 触发表单验证
        this.form.validateFields((err, values) => {
          if (!err) {
            that.confirmLoading = true;
//            postAction(httpurl,formData,method).then((res)=>{
//              if(res.success){
//                that.$message.success(res.message);
//                that.$emit('ok');
//              }else{
//                that.$message.warning(res.message);
//              }
//            }).finally(() => {
//              that.confirmLoading = false;
//              that.close();
//            })
          }
        })
      },
      selectedCategory(selectedKeys,e){
        this.queryParam.productType = e.node.dataRef.code;
        this.searchQuery();
      },
      searchQuery() {
        this.loadData(1);
      },
      searchReset() {
        this.queryParam = {}
        this.$refs.categoryTree.selectedKeys = [];
        this.loadData(1);
      },

    }
  }
</script>

<style lang="less" scoped>

</style>