<template>
  <a-modal
    title="设置价格库存"
    :width="900"
    :visible="visible"
    :confirmLoading="confirmLoading"
    @ok="handleOk"
    @cancel="handleCancel"
    cancelText="关闭">
    <div>
      <a-table bordered :dataSource="dataSource" :columns="columns">
        <template slot="name" slot-scope="text, record">
          <editable-cell :text="text" @change="onCellChange(record.key, 'name', $event)"/>
        </template>
      </a-table>
    </div>
  </a-modal>
</template>
<script>
  import EditableCell from '@/views/ufak/common/EditableCell'
  import {getAction} from '@/api/manage'

  export default {
    components: {
      EditableCell,
    },
    data () {
      return {
        visible: false,
        confirmLoading: false,
        dataSource: [],
        productSpecsList: [],
        columns: [{
          title: '规格名称',
          dataIndex: 'name',
          width: '30%',
          scopedSlots: {customRender: 'name'},
        }, {
          title: '虚拟价',
          dataIndex: 'virtualPrice',
          width: '17.5%',
        }, {
          title: '折扣',
          dataIndex: 'discount',
          width: '17.5%',
        }, {
          title: '售卖价',
          dataIndex: 'price',
          width: '17.5%',
        }, {
          title: '库存',
          dataIndex: 'stock',
          width: '17.5%',
        }],
        url: {
          save: "/product/info/add",
          getProductSpecsList: "/product/specs/list",
        },
      }
    },
    created() {
    },
    methods: {
//      onCellChange (key, dataIndex, value) {
//        const dataSource = [...this.dataSource]
//        const target = dataSource.find(item => item.key === key)
//        if (target) {
//          target[dataIndex] = value
//          this.dataSource = dataSource
//        }
//      },
      edit(record) {
        let params = {
          productId: record.id,
          orderBy: 'sort',
          pageNo: 1,
          pageSize: 1000
        };
        getAction(this.url.getProductSpecsList, params).then((res) => {
          if (res.success) {
            this.productSpecsList = res.result.records;
          }
        });

      },
      close() {
        this.$emit('close');
        this.visible = false;
      },
      handleOk() {

      },
      handleCancel() {
        this.close()
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
