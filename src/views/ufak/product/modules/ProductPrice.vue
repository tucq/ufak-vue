<template>
  <a-modal
    title="设置价格库存"
    :width="800"
    :visible="visible"
    :maskClosable="false"
    :confirmLoading="confirmLoading"
    @ok="handleOk"
    @cancel="handleCancel"
    cancelText="关闭">
    <div style="margin-top: -10px;">
      <a-row>
        <a-col :span="22"><span style="font-size: 14px;">商品名称：<span style="color: #1890ff">{{productName}}</span></span></a-col>
        <a-col :span="2"><a-button type="primary" @click="tableEdit" size="small" icon="edit">编辑</a-button></a-col>
      </a-row>
      <a-radio-group name="radioGroup" @change="radioChange" style="width: 100%" v-model="radioValue">
      <a-table style="margin-top: 10px;" bordered rowKey="id" :dataSource="dataSource" :columns="columns"
               :pagination="ipagination" @change="handleTableChange">
        <template v-for="col in ['virtualPrice','price','stock', 'defaultFlag']"
                  :slot="col" slot-scope="text, record, index"
                  :indentSize="30">
          <div :key="col">
            <div v-if="col == 'defaultFlag'">
              <a-radio name="abc" :value="record.id" />
            </div>
            <div v-else>
              <a-input
                style="margin: -5px 0"
                v-if="isEdit"
                :value="text"
                @change="e => onCellChange( record.id, col,e.target.value)"
              />
              <template v-else>{{text}}</template>
            </div>
<!--            <editableCell :text="text" @change="onCellChange(record.id, col, $event)" :parentEditable="parentEditable"/>-->
          </div>
        </template>
      </a-table>
      </a-radio-group>
    </div>
  </a-modal>
</template>
<script>
  import EditableCell from '@/views/ufak/common/EditableCell'
  import { filterObj } from '@/utils/util'
  import {putAction,getAction,postAction} from '@/api/manage'
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
        isEdit: false,
        radioValue: '',
        columns: [
        {
            title: '默认',
            dataIndex: 'defaultFlag',
            width: '10%',
            scopedSlots: {customRender: 'defaultFlag'},
        },{
          title: '规格名称',
          dataIndex: 'specsName',
          width: '30%',
        }, {
          title: '虚拟价',
          dataIndex: 'virtualPrice',
          width: '20%',
          scopedSlots: {customRender: 'virtualPrice'},
        }, {
          title: '售卖价',
          dataIndex: 'price',
          width: '20%',
          scopedSlots: {customRender: 'price'},
        }, {
          title: '库存',
          dataIndex: 'stock',
          width: '20%',
          scopedSlots: {customRender: 'stock'},
        }],
        ipagination:{
          current: 1,
          pageSize: 100,
          pageSizeOptions: ['100'],
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
          // 校验整数
          if(dataIndex === 'stock'){
              let s = /^\d+(\.\d+)?$/;
              if(!s.test(value)){
                  value = "";
              }
          }
          // 校验金额
          // if(dataIndex === 'virtualPrice' || dataIndex === 'price'){
          //     // let s = /(^[1-9](\d+)?(\.\d{1,2})?$)|(^[1-9]$)|(^\d\.[1-9]{1,2}$)|(^\d\.[0]{1}[1-9]{1}$|(^\d\.[1-9]{1}[0]{1}$)|(.)$)/;
          //     if(!s.test(value)){
          //         value = "";
          //     }
          // }

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
        //要提交的数据先重置
        this.radioValue = '';
        this.updatePrice = [];

        if(type){
            this.isEdit = true;
        }else{
            this.isEdit = false;
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
          let formData = {
              "productPriceList": this.updatePrice,
              "defaultFlag": {"productId": this.queryParam.productId,"id" : this.radioValue},
          };
          console.log("formData",formData);
          putAction(this.url.edit, formData).then((res) => {
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
            //设置默认选项
            for(let i=0;i<this.dataSource.length;i++){
                if(this.dataSource[i].defaultFlag == '0'){
                    this.radioValue = this.dataSource[i].id;
                }
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
      handleTableChange(pagination, filters, sorter) {
        if (Object.keys(sorter).length > 0) {
          this.isorter.column = sorter.field;
          this.isorter.order = "ascend" == sorter.order ? "asc" : "desc"
        }
        this.ipagination = pagination;
        this.loadData();
      },
      tableEdit(){
         this.isEdit = !this.isEdit;
      },
      radioChange(e){
          this.radioValue = e.target.value;
      },

    },
  }
</script>
<style>

</style>
