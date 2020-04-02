<template>
  <a-row :gutter="10">
    <a-col :md="7" :sm="24">
      <a-card :bordered="false">
        <!-- 查询区域 -->
        <div class="table-page-search-wrapper">
          <a-form layout="inline" @keyup.enter.native="searchQuery">
            <a-row :gutter="24">
              <a-col :md="12" :sm="16">
                <a-form-item label="姓名">
                  <a-input placeholder="请输姓名" v-model="queryParam.username"></a-input>
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
            @change="handleTableChange"
            :customRow="onClickRow">
            <span slot="avatar" slot-scope="avatar">
                <a-row>
                  <img :src="avatar" style="height:50px;max-width:50px"/>
                </a-row>
            </span>
          </a-table>
        </div>

      </a-card>
    </a-col>
    <a-col :md="17" :sm="24">
      <a-card :bordered="false">
        <div class="table-operator">
          <span style="padding-left: 20px;">
            微信用户：<span style="color: red;">{{realname}}</span>
          </span>
        </div>

        <!-- table区域-begin -->
        <div>
          <a-table
            ref="table"
            size="middle"
            bordered
            rowKey="id"
            :columns="addressColumns"
            :dataSource="addressDataSource"
            :pagination="addressPagination"
            :loading="loading"
          >
            <span slot="addressAction" slot-scope="text, record">
              <a @click="handleView(record)">查看</a>
            </span>
          </a-table>
        </div>
        <userAddress-modal ref="modalForm" @ok="modalFormOk"></userAddress-modal>
      </a-card>
    </a-col>
  </a-row>

</template>

<script>
  import UserAddressModal from './modules/UserAddressModal'
  import {JeecgListMixin} from '@/mixins/JeecgListMixin'
  import { getAction } from '@/api/manage'
  import { filterObj } from '@/utils/util'

  export default {
    name: "ShoppingCarList",
    mixins: [JeecgListMixin],
    components: {
      UserAddressModal
    },
    data () {
      return {
        description: '购物车',
        userId: "",
        realname:'',
        // 表头
        columns: [
          {
            title: '#',
            dataIndex: '',
            key: 'rowIndex',
            width: "10%",
            align: "center",
            customRender: function (t, r, index) {
              return parseInt(index) + 1;
            }
          },
          {
            title: '头像',
            align: "center",
            width: "30%",
            dataIndex: 'avatar',
            scopedSlots: {customRender: 'avatar'},
          },
          {
              title: '微信名称',
              align: "center",
              width: "60%",
              dataIndex: 'realname'
          },
        ],
        addressDataSource:[],
        addressColumns: [
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
            title: '收货姓名',
            width: '20%',
            align: "center",
            dataIndex: 'username',
          },
          {
            title: '收货电话号码',
            width: '30%',
            align: "center",
            dataIndex: 'telephone'
          },
          {
            title: '地区',
            width: '18%',
            align: "center",
            dataIndex: 'address',
          },
          {
            title: '详细地址',
            width: '10%',
            align: "center",
            dataIndex: 'detailAddress'
          },
          {
            title: '操作',
            width: '17%',
            dataIndex: 'action',
            align: "center",
            scopedSlots: {customRender: 'addressAction'},
          }
        ],
        addressPagination:{
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
          list: "/usr/address/userList",
          addressList:"/usr/address/list",
        },
      }
    },
    created() {
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

            //加载完后查询地址信息
            if(this.dataSource.length > 0){
              let user = this.dataSource[0];
              this.userId = user.id;
              this.realname = user.realname;
              this.loadAddress(1);
            }
          }
        })
      },
      getQueryParams(){
        let param = Object.assign({}, this.queryParam,this.isorter);
        param.pageNo = this.ipagination.current;
        param.pageSize = this.ipagination.pageSize;
        return filterObj(param);
      },
      searchQuery(){
        this.loadData(1);
      },
      searchReset(){
        this.queryParam={};
        this.loadData(1);
      },

      loadAddress(arg){
        if(arg===1){
          this.addressPagination.current = 1;
        }
        getAction(this.url.addressList,{userId: this.userId}).then((res)=>{
          if(res.success){
            this.addressDataSource = res.result.records;
            this.addressPagination.total = res.result.total;
          }
        })
      },
      onClickRow (record) {
        return {
          on: {
            click: () => {
              this.userId = record.id;
              this.realname = record.realname;
              this.loadAddress(1);
            }
          }
        }
      },

    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less'
</style>