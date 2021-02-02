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
          label="反馈人">
          <a-input placeholder="请输入反馈人" v-decorator="['username', {}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="联系电话">
          <a-input placeholder="请输入联系电话" v-decorator="['telephone', {}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="内容">
          <a-textarea :rows="5" placeholder="请输入内容" v-decorator="['content', {}]" />
        </a-form-item>

        <div style="padding: 10px;">
          <a-row :gutter="10">
            <a-col :span="6" v-for="(img,idx) in images">
              <a-card :bordered="false" @click="handlePreview(img)">
                <img :src="getAvatarView(img)" style="height:150px;max-width:150px" />
              </a-card>
            </a-col>
          </a-row>
        </div>
        <a-modal :visible="previewVisible" :footer="null" @cancel="imageCancel">
          <img alt="example" style="width: 100%" :src="previewImage"/>
        </a-modal>
		
      </a-form>
    </a-spin>
  </a-modal>
</template>

<script>
  import { httpAction } from '@/api/manage'
  import pick from 'lodash.pick'
  import moment from "moment"
  import ATextarea from "ant-design-vue/es/input/TextArea";

  export default {
    components: {ATextarea}, name: "FeedBackModal",
    data () {
      return {
        title:"操作",
        visible: false,
        previewVisible: false,
        previewImage: '',
        images: [],
        model: {},
        labelCol: {
          xs: { span: 24 },
          sm: { span: 5 },
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 16 },
        },

        confirmLoading: false,
        form: this.$form.createForm(this),
        validatorRules:{
        },
        url: {
          add: "/feedBack/add",
          edit: "/feedBack/edit",
        },
      }
    },
    created () {
    },
    methods: {
      getAvatarView(image){
        return window._CONFIG['domianURL']+"/sys/common/view/"+ image;
      },
      imageCancel() {
        this.previewVisible = false;
      },
      handlePreview(url) {
        this.previewImage = window._CONFIG['domianURL']+"/sys/common/view/"+ url;
        this.previewVisible = true;
      },
      add () {
        this.edit({});
      },
      edit (record) {
        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.images = record.images.split(',');
        this.visible = true;
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model,'content','images','username','telephone'))
		  //时间格式化
        });

      },
      close () {
        this.$emit('close');
        this.visible = false;
      },
      handleOk () {
        const that = this;
        // 触发表单验证
        this.form.validateFields((err, values) => {
          if (!err) {
            that.confirmLoading = true;
            let httpurl = '';
            let method = '';
            if(!this.model.id){
              httpurl+=this.url.add;
              method = 'post';
            }else{
              httpurl+=this.url.edit;
               method = 'put';
            }
            let formData = Object.assign(this.model, values);
            //时间格式化
            
            console.log(formData)
            httpAction(httpurl,formData,method).then((res)=>{
              if(res.success){
                that.$message.success(res.message);
                that.$emit('ok');
              }else{
                that.$message.warning(res.message);
              }
            }).finally(() => {
              that.confirmLoading = false;
              that.close();
            })



          }
        })
      },
      handleCancel () {
        this.close()
      },


    }
  }
</script>

<style lang="less" scoped>

</style>