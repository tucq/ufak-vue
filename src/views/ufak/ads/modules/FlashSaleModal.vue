<template>
  <a-modal
    :title="title"
    :width="800"
    :visible="visible"
    :confirmLoading="confirmLoading"
    @ok="handleOk"
    @cancel="handleCancel"
    cancelText="关闭">

    <a-spin :spinning="confirmLoading">
      <a-form :form="form">
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="抢购标题">
          <a-input placeholder="请输入标题" v-decorator="['title', validatorRules.title]"/>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="抢购描述">
          <a-input placeholder="请输入描述" v-decorator="['remark',validatorRules.remark]"/>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="秒杀价">
          <a-input v-decorator="[ 'killPrice', validatorRules.killPrice]"/>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="抢购数量">
          <a-input v-decorator="[ 'stock', validatorRules.stock]"/>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="开始时间">
          <!--<a-date-picker style="width:100%" showTime format='YYYY-MM-DD HH:mm:ss'  />-->
          <a-time-picker style="width:100%" :default-open-value="moment('00:00:00', 'HH:mm:ss')"
                         v-decorator="[ 'startTime', validatorRules.startTime]" @change="onChangeStartTime"/>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="结束时间">
          <!--<a-date-picker style="width:100%" showTime format='YYYY-MM-DD HH:mm:ss' v-decorator="[ 'endTime', validatorRules.endTime]" />-->
          <a-time-picker style="width:100%" :default-open-value="moment('00:00:00', 'HH:mm:ss')"
                         v-decorator="[ 'endTime', validatorRules.endTime]" @change="onChangeEndTime"/>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="状态">
          <a-row>
            <a-col :span="12">
              <a-switch checkedChildren="启用" unCheckedChildren="停用" :checked="state == '0' ? true : false"
                        @click="e => handelCheck(e)"/>
            </a-col>
            <a-col :span="8" push="6">
              <a-button @click="refProduct" type="primary" icon="plus">关联商品</a-button>
            </a-col>
            <a-col :span="4" push="4">
              <a-tooltip placement="top">
                <template slot="title">
                  <span>关联的商品为下架的商品</span>
                </template>
                <a-icon type="question-circle"/>
              </a-tooltip>
            </a-col>
          </a-row>
        </a-form-item>

        <!--<a-form-item>-->
        <!--<a-row>-->
        <!--<a-col>-->
        <!--<img-->
        <!--alt="图片"-->
        <!--:src="previewUrl('files/20201219/主图1_1608369766941.jpg')"-->
        <!--style="height:50px;max-width:50px"-->
        <!--slot="cover"-->
        <!--/>-->
        <!--</a-col>-->
        <!--<a-col>-->
        <!--xxxx-->
        <!--</a-col>-->
        <!--</a-row>-->
        <!--</a-form-item>-->


      </a-form>
    </a-spin>
    <flash-sale-product ref="flashSaleProduct" @ok="selectedProduct"></flash-sale-product>
  </a-modal>

</template>

<script>
  import {httpAction} from '@/api/manage'
  import pick from 'lodash.pick'
  import moment from "moment"
  import ARow from "ant-design-vue/es/grid/Row";
  import ACol from "ant-design-vue/es/grid/Col";
  import FlashSaleProduct from './FlashSaleProduct.vue'

  export default {
    components: {ACol, ARow, FlashSaleProduct}, name: "FlashSaleModal",
    data() {
      return {
        title: "操作",
        visible: false,
        state: '0',
        productId: '',
        model: {},
        labelCol: {
          xs: {span: 24},
          sm: {span: 5},
        },
        wrapperCol: {
          xs: {span: 24},
          sm: {span: 16},
        },

        confirmLoading: false,
        form: this.$form.createForm(this),
        validatorRules: {
          title: {rules: [{required: true, message: '请输入限时抢购标题'}]},
          remark: {rules: [{required: true, message: '请输入限时抢购描述'}]},
          killPrice: {rules: [{required: true, message: '请输入秒杀价'}]},
          stock: {rules: [{required: true, message: '请输入抢购数量'}]},
          state: {rules: [{required: true, message: '请输入状态'}]},
          startTime: {rules: [{required: true, message: '请输入开始时间'}]},
          endTime: {rules: [{required: true, message: '请输入结束时间'}]},
        },
        url: {
          add: "/flashSale/add",
          edit: "/flashSale/edit",
        },
      }
    },
    created() {
    },
    methods: {
      moment,
      add() {
        this.edit({state: '0', startTime: '10:00:00', endTime: '11:00:00'});
      },
      edit(record) {
        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.productId = record.productId;
        this.visible = true;
        console.log("this.model", this.model);
        this.model.startTime = moment(record.startTime, 'HH:mm:ss');
        this.model.endTime = moment(record.endTime, 'HH:mm:ss');
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model, 'productId', 'title', 'remark', 'killPrice', 'stock', 'state', 'startTime', 'endTime'))
        });
        this.state = record.state;

      },
      close() {
        this.$emit('close');
        this.visible = false;
      },
      handleOk() {
        const that = this;
        // 触发表单验证
        this.form.validateFields((err, values) => {
          if (!err) {
            that.confirmLoading = true;
            let httpurl = '';
            let method = '';
            if (!this.model.id) {
              httpurl += this.url.add;
              method = 'post';
            } else {
              httpurl += this.url.edit;
              method = 'put';
            }
            let formData = Object.assign(this.model, values);
            //时间格式化
            formData.startTime = formData.startTime ? formData.startTime.format('HH:mm:ss') : null;
            formData.endTime = formData.endTime ? formData.endTime.format('HH:mm:ss') : null;
            formData.productId = this.productId;
            formData.state = this.state;
            console.log(formData);

            if (!this.productId) {
              that.$message.error("没有关联商品");
              that.confirmLoading = false;
              return;
            }
            httpAction(httpurl, formData, method).then((res) => {
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
        })
      },
      handleCancel() {
        this.close()
      },
      handelCheck(check) {
        if (check) {
          this.state = '0';
        } else {
          this.state = '1';
        }
      },
      selectedProduct(productIds) {
        this.productId = productIds[0];
      },
      refProduct() {
        this.$refs.flashSaleProduct.title = "关联秒杀商品";
        this.$refs.flashSaleProduct.visible = true;
      },
      onChangeStartTime(time, timeString) {
        this.model.startTime = timeString;
      },
      onChangeEndTime(time, timeString) {
        this.model.endTime = timeString;
      },

    }
  }
</script>

<style lang="less" scoped>

</style>