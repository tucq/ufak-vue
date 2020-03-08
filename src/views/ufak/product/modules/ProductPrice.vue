<template>
  <a-modal
    title="设置价格库存"
    :width="900"
    :visible="visible"
    :confirmLoading="confirmLoading"
    @ok="handleOk"
    @cancel="handleCancel"
    cancelText="关闭">
    <div style="margin-top: -10px;">
      <span style="font-size: 14px;">商品名称：<span style="color: #1890ff">{{productName}}</span></span>
      <a-table style="margin-top: 10px;" bordered rowKey="id" :dataSource="dataSource" :columns="columns"
               :pagination="ipagination" @change="handleTableChange">
        <template v-for="col in ['virtualPrice', 'discount','price','stock','stats']"
                  :slot="col" slot-scope="text, record, index"
                  :indentSize="30">
          <div :key="col">
            <editableCell :text="text" @change="onCellChange(record.id, col, $event)" :parentEditable="parentEditable"/>
          </div>
        </template>
      </a-table>
    </div>
  </a-modal>
</template>
<script>
  import EditableCell from '@/views/ufak/common/EditableCell'
  import { filterObj } from '@/utils/util'
  import {putAction,getAction} from '@/api/manage'
  import ARow from "ant-design-vue/es/grid/Row";

  export default {
    components: {
      ARow, EditableCell,
    },
    data () {
      return {
        visible: false,
        confirmLoading: false,
        queryParam: {},
        dataSource: [],
        updatePrice: [],
        productName: "",
        parentEditable: false,
        columns: [{
          title: '规格名称',
          dataIndex: 'specsName',
          width: '30%',
        }, {
          title: '虚拟价',
          dataIndex: 'virtualPrice',
          width: '17.5%',
          scopedSlots: {customRender: 'virtualPrice'},
        }, {
          title: '折扣',
          dataIndex: 'discount',
          width: '17.5%',
          scopedSlots: {customRender: 'discount'},
        }, {
          title: '售卖价',
          dataIndex: 'price',
          width: '17.5%',
          scopedSlots: {customRender: 'price'},
        }, {
          title: '库存',
          dataIndex: 'stock',
          width: '17.5%',
          scopedSlots: {customRender: 'stock'},
        }],
        ipagination:{
          current: 1,
          pageSize: 7,
          pageSizeOptions: ['10', '20', '30'],
          showTotal: (total, range) => {
            return range[0] + "-" + range[1] + " 共" + total + "条"
          },
          showQuickJumper: true,
          showSizeChanger: true,
          total: 0
        },
        url: {
          edit: "/product/price/update/price",
          list: "/product/price/list",
        },
      }
    },
    created() {
    },
    methods: {
      onCellChange (id, dataIndex, value) {
        const dataSource = [...this.dataSource]
        const target = dataSource.find(item => item.id === id);
        if (target) {
          target[dataIndex] = value
          this.dataSource = dataSource
          let isExit = false;
          for(let i=0;i<this.updatePrice.length;i++){
            if(this.updatePrice[i].id == target.id){// 存在更新
                this.updatePrice[i] = target;
                isExit = true;
            }
          }
          if(!isExit){ //不存在仍进去
            this.updatePrice.push(target);
          }

          console.log(this.updatePrice);
        }

      },

      edit(record,type) {
        if(type){
            this.parentEditable = true;
        }else{
            this.parentEditable = false;
        }
        this.productName = record.name;
        this.queryParam.productId = record.id;
        this.loadData(1);
      },
      close() {
        this.$emit('close');
        this.visible = false;
      },
      handleOk() {
          const that = this;
          putAction(this.url.edit, this.updatePrice).then((res) => {
            if (res.success) {
              that.$message.success(res.message);
              that.$emit('ok');
            } else {
              that.$message.warning(res.message);
            }
          }).finally(() => {
            that.confirmLoading = false;
            that.close();
          })
      },
      handleCancel() {
        this.close()
      },
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
      getQueryParams(){
        let param = Object.assign({}, this.queryParam,this.isorter);
        param.pageNo = this.ipagination.current;
        param.pageSize = this.ipagination.pageSize;
        return filterObj(param);
      },
      handleTableChange(pagination, filters, sorter) {
        if (Object.keys(sorter).length > 0) {
          this.isorter.column = sorter.field;
          this.isorter.order = "ascend" == sorter.order ? "asc" : "desc"
        }
        this.ipagination = pagination;
        this.loadData();
      },

    },
  }
</script>
<style>
  .editable-cell {
    position: relative;
  }

  .editable-cell-input-wrapper,
  .editable-cell-text-wrapper {
    padding-right: 24px;
  }

  .editable-cell-text-wrapper {
    padding: 5px 24px 5px 5px;
  }

  .editable-cell-icon,
  .editable-cell-icon-check {
    position: absolute;
    right: 0;
    width: 20px;
    cursor: pointer;
  }

  .editable-cell-icon {
    line-height: 18px;
    display: none;
  }

  .editable-cell-icon-check {
    line-height: 28px;
  }

  .editable-cell:hover .editable-cell-icon {
    display: inline-block;
  }

  .editable-cell-icon:hover,
  .editable-cell-icon-check:hover {
    color: #108ee9;
  }

  .editable-add-btn {
    margin-bottom: 8px;
  }
</style>
