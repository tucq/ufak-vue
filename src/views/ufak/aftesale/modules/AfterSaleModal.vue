<template>
  <a-modal
    :title="title"
    :width="800"
    style="top: 0px;"
    :visible="visible"
    :confirmLoading="confirmLoading"
    :okButtonProps="{ props: {disabled: disableSubmit} }"
    okText="退款"
    @ok="handleOk"
    @cancel="handleCancel"
    cancelText="关闭">

    <a-spin :spinning="confirmLoading">
      <a-row :gutter="20" style="font-size: 16px;font-weight: bold;padding-left: 30px;">退款信息</a-row>
      <a-form>
        <a-row :gutter="20">
          <a-col :span="12">
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="微信单号">
              {{model.transactionId}}
            </a-form-item>
          </a-col>
          <a-col :span="12">
          </a-col>
        </a-row>
        <a-row :gutter="20">
          <a-col :span="12">
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="退款单号">
              {{model.afterSaleNo}}
            </a-form-item>
          </a-col>
          <a-col :span="12">
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="退款时间">
              {{model.createTime}}
            </a-form-item>
          </a-col>
        </a-row>
        <a-row :gutter="20">
          <a-col :span="12">
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="退款人">
              {{model.refundContact}}
            </a-form-item>
          </a-col>
          <a-col :span="12">
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="联系电话">
              {{model.refundTelephone}}
            </a-form-item>
          </a-col>
        </a-row>
        <a-row :gutter="20">
          <a-col :span="12">
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="退款原因">
              {{model.refundReason}}
            </a-form-item>
          </a-col>
          <a-col :span="12">
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="退款金额">
              {{model.paymentAmount}}
            </a-form-item>
          </a-col>
        </a-row>
        <a-row :gutter="20">
          <a-col :span="12">
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="状态">
              {{model.statusText}}
            </a-form-item>
          </a-col>
          <a-col :span="12">
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="备注">
              {{model.remark}}
            </a-form-item>
          </a-col>
        </a-row>
        <a-row :gutter="20" style="font-size: 16px;font-weight: bold;padding-left: 30px;">订单信息</a-row>
        <a-row :gutter="20">
          <a-col :span="12">
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="订单编号">
              {{order.orderNo}}
            </a-form-item>
          </a-col>
          <a-col :span="12">
          </a-col>
        </a-row>
        <a-row :gutter="20">
          <a-col :span="12">
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="订单总金额">
              {{order.totalAmount}}
            </a-form-item>
          </a-col>
          <a-col :span="12">
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="运费">
              {{order.freightAmount}}
            </a-form-item>
          </a-col>
        </a-row>
        <a-row :gutter="20">
          <a-col :span="12">
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="活动优惠券">
              {{order.eventAmount}}
            </a-form-item>
          </a-col>
          <a-col :span="12">
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="实付金额">
              {{order.paymentAmount}}
            </a-form-item>
          </a-col>
        </a-row>
        <a-row :gutter="20" style="padding-top: 10px;" v-for="(item, index) in orderDetails" :key="index">
          <a-col :span="6" push="1">
            <a-row :gutter="20" type="flex" justify="start" align="middle">
              <img
                :src="previewUrl(item.viewImage)"
                style="height:100px;width:100px"
              />
            </a-row>
          </a-col>
          <a-col :span="18">
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
  import {getAction, postAction} from '@/api/manage'
  import pick from 'lodash.pick'
  import moment from "moment"

  export default {
    name: "AfterSaleModal",
    data() {
      return {
        title: "操作",
        visible: false,
        disableSubmit: false,
        model: {},
        order: {},
        orderDetails: [],
        labelCol: {
          xs: {span: 24},
          sm: {span: 7},
        },
        wrapperCol: {
          xs: {span: 24},
          sm: {span: 16},
        },
        confirmLoading: false,
        validatorRules: {
          orderId: {rules: [{required: true, message: '请输入订单id!'}]},
        },
        url: {
          queryById: "/afterSaleRefund/queryById",
          wxRefund: "/refund/wxRefund",
        },
      }
    },
    created() {
    },
    methods: {
      loadData(record) {
        getAction(this.url.queryById, {"afterSaleId": record.id}).then((res) => {
          if (res.success) {
            this.model = res.result.afterSaleRefund;
            this.order = res.result.order;
            this.orderDetails = res.result.orderDetails;
            this.visible = true;
          }
        })
      },
      close() {
        this.$emit('close');
        this.visible = false;
      },
      handleOk() {
        const that = this;
        this.$confirm({
          title: "确认退款是否提交",
          content: "提交后退款金额会原路退回，请再次确认该笔订单是否已发货?",
          onOk: function () {
            that.confirmLoading = true;
            let params = {
              "afterSaleId": that.model.afterSaleId
            }
            postAction(that.url.wxRefund, params).then((res) => {
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
          }
        });
      },
      handleCancel() {
        this.close()
      },
      previewUrl(url) {
        return window._CONFIG['domianURL'] + "/sys/common/view/" + url;
      }


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