<template>
  <a-modal
    :title="title"
    :width="600"
    :visible="visible"
    :maskClosable="false"
    :confirmLoading="confirmLoading"
    :bodyStyle="{'padding-top': '0px'}"
    @ok="handleOk"
    @cancel="handleCancel"
    cancelText="关闭">
    <a-spin :spinning="confirmLoading">
      <a-layout>
        <a-layout-content style="background:#ffffff;">
          <div class="table-page-search-wrapper" style="padding-top:20px;">
            <a-form layout="inline">
              <a-row :gutter="24">
                <a-col :span="14">
                  <a-form-item label="商品名称">
                    <a-input v-model="queryParam.productName" placeholder="请输入商品名称" @pressEnter="searchQuery"/>
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
            :rowSelection="{type:'radio',selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"
            @change="handleTableChange"
            >
            <span slot="image" slot-scope="image">
                <a-row>
                  <img :src="getAvatarView(image.split(',')[0])" style="height:50px;max-width:50px"/>
                </a-row>
            </span>
          </a-table>
        </a-layout-content>
      </a-layout>
    </a-spin>
  </a-modal>
</template>

<script>
  import { postAction,getAction} from '@/api/manage'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'

  export default {
    mixins:[JeecgListMixin],
    components: {
    },
    data () {
      return {
        title:"操作",
        visible: false,
        adsId: "",
        queryParam:{},
        dataSource:[],
        selectedRowKeys:[],
        confirmLoading: false,
        columns: [
          {
            title: '商品图片',
            width: '30%',
            align: "center",
            dataIndex: 'image',
            scopedSlots: {customRender: 'image'},
          },
          {
            title: '商品名称',
            width: '65%',
            align: "center",
            dataIndex: 'name'
          },
        ],
        ipagination: {
          current: 1,
          pageSize: 5,
          pageSizeOptions: ['5', '10', '15'],
          showTotal: (total, range) => {
            return range[0] + '-' + range[1] + ' 共' + total + '条'
          },
          showQuickJumper: true,
          showSizeChanger: true,
          total: 0
        },
        url: {
          list: "/product/info/list",
          imgerver: window._CONFIG['domianURL']+"/sys/common/view",
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
        params.state = '0';
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
        that.confirmLoading = true;
        if(this.selectedRowKeys.length == 0){
          that.$message.error("请选择商品");
          that.confirmLoading = false;
          return;
        }
        that.$message.success("商品关联成功");
        that.$emit('ok',this.selectedRowKeys);
        that.confirmLoading = false;
        that.close();
      },
      getAvatarView(image){
        return this.url.imgerver +"/"+ image;
      },

      searchQuery() {
        this.loadData(1);
      },
      searchReset() {
        this.queryParam = {}
        this.loadData(1);
      },

    }
  }
</script>

<style lang="less" scoped>

</style>