<template>
  <a-modal
    :title="title"
    :width="1200"
    style="top: 0px;"
    :visible="visible"
    :confirmLoading="confirmLoading"
    @ok="handleOk"
    @cancel="handleCancel"
    cancelText="关闭">

    <a-spin :spinning="confirmLoading">
      <a-form :form="form">

        <a-row :gutter="20">
          <a-col :span="8">
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="订单编号">
              {{model.orderNo}}
            </a-form-item>
          </a-col>
          <a-col :span="8">
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="实收金额">
              {{model.paymentAmount}}
            </a-form-item>
          </a-col>
          <a-col :span="8">
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="物流编号">
              {{model.logisticsNo}}
            </a-form-item>
          </a-col>
        </a-row>

        <a-row :gutter="20">
          <a-col :span="8">
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="订单总金额">
              {{model.totalAmount}}
            </a-form-item>
          </a-col>
          <a-col :span="8">
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="运费">
              {{model.freightAmount}}
            </a-form-item>
          </a-col>
          <a-col :span="8">
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="活动优惠券">
              {{model.eventAmount}}
            </a-form-item>
          </a-col>
        </a-row>

        <a-row :gutter="20">
          <a-col :span="8">
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="优惠券">
              {{model.couponAmount}}
            </a-form-item>
          </a-col>
          <a-col :span="8">
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="订单状态">
              {{model.orderStatus}}
            </a-form-item>
          </a-col>
          <a-col :span="8">
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="收货人/电话">
              {{model.username}}&nbsp;&nbsp;{{model.telephone}}
            </a-form-item>
          </a-col>
        </a-row>

        <a-row :gutter="20">
          <a-col :span="8">
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="地区">
              {{model.address}}
            </a-form-item>
          </a-col>
          <a-col :span="8">
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="微信支付订单号">
              {{model.transactionId}}
            </a-form-item>
          </a-col>
          <a-col :span="8">
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="创建时间">
              {{model.createTime}}
            </a-form-item>
          </a-col>
        </a-row>

        <a-row :gutter="20">
          <a-col :span="24" style="padding-left: 23px;">
            <a-form-item
              :labelCol="{xs: { span: 24 },sm: { span: 2 },offset: 0}"
              :wrapperCol="{xs: { span: 24 },sm: { span: 14}}"
              label="详细地址">
              {{model.detailAddress}}
            </a-form-item>
          </a-col>
        </a-row>

        <a-row :gutter="20" style="padding-top: 10px;" v-for="(item, index) in model.orderDetails" :key="index">
          <a-col :span="4" push="1">
            <a-row :gutter="20" type="flex" justify="start" align="middle">
              <img
                :src="previewUrl(item.viewImage)"
                style="height:100px;width:100px"
              />
            </a-row>
          </a-col>
          <a-col :span="20">
            <a-row :gutter="20" type="flex" justify="start" align="middle" style="height:30px;">
              {{item.productName}}
            </a-row>
            <a-row :gutter="20" type="flex" justify="start" align="middle" style="height:30px;">
              {{item.specsName}}&nbsp;&nbsp;&nbsp;&nbsp;x&nbsp;{{item.buyNum}}
            </a-row>
            <a-row :gutter="20" type="flex" justify="start" align="middle" style="height:30px;">
              {{item.price}}
            </a-row>
          </a-col>
        </a-row>


      </a-form>
    </a-spin>
  </a-modal>
</template>

<script>
  import {getAction} from '@/api/manage'
  import pick from 'lodash.pick'
  import moment from "moment"

  export default {
    name: "OrderModal",
    data() {
      return {
        title: "操作",
        visible: false,
        model: {},
        orderDetails: [],
        labelCol: {
          xs: {span: 24},
          sm: {span: 7},
        },
        wrapperCol: {
          xs: {span: 24},
          sm: {span: 14},
        },
        confirmLoading: false,
        form: this.$form.createForm(this),
        validatorRules: {},
        url: {},
      }
    },
    created() {
    },
    methods: {
      add() {
        this.edit({});
      },
      edit(record) {
//        this.form.resetFields();
//        this.model = Object.assign({}, record);
//        this.visible = true;
//        this.$nextTick(() => {
//          this.form.setFieldsValue(pick(this.model,'userId','orderNo','logisticsNo','orderAmount','eventAmount','couponAmount','paymentAmount','orderStatus','username','telephone','address','detailAddress','transactionId','sign','totalFee','cashFee','resultCode'))
//		  //时间格式化
//        });

        getAction('/order/' + record.id, null).then((res) => {
          if (res.success) {
            this.model = res.result;
            this.visible = true;
          }
        })
      },
      previewUrl(url) {
        return window._CONFIG['domianURL'] + "/sys/common/view/" + url;
      },
      close() {
        this.$emit('close');
        this.visible = false;
      },
      handleOk() {
        this.close();
      },
      handleCancel() {
        this.close()
      },


    }
  }
</script>

<style lang="less" scoped>

  .ant-form-item {
    box-sizing: border-box;
    margin: 0;
    padding: 0;
    color: rgba(0, 0, 0, 0.65);
    font-size: 14px;
    font-variant: tabular-nums;
    line-height: 1.5;
    list-style: none;
    font-feature-settings: 'tnum';
    margin-bottom: 0px;
    vertical-align: top;
  }

</style>