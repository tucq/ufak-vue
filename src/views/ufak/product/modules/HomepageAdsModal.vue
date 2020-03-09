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
          label="广告类型">
          <j-dict-select-tag v-decorator="['type', {}]" :triggerChange="true" placeholder="请输入广告类型" dictCode="ads_type"/>
        </a-form-item>
        <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="广告图片">
          <!--<a-input placeholder="请输入广告图片" v-decorator="['imgUrl', {}]" />-->
          <a-upload
            :action="uploadAction"
            listType="picture-card"
            :fileList="fileList"
            :headers="headers"
            :beforeUpload="beforeUpload"
            :remove="imageRemove"
            @preview="handlePreview"
            @change="imageChange"
          >
            <div v-if="fileList.length < 1">
              <a-icon type="plus"/>
              <div class="ant-upload-text">上 传</div>
            </div>
          </a-upload>
          <a-modal :visible="previewVisible" :footer="null" @cancel="imageCancel">
            <img alt="example" style="width: 100%" :src="previewImage"/>
          </a-modal>
        </a-form-item>

        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="详情头部图片">
          <a-input placeholder="请输入头部广告" v-decorator="['headImg', {}]" />
          <!--<a-upload-->
            <!--:action="uploadAction"-->
            <!--listType="picture-card"-->
            <!--:fileList="fileList"-->
            <!--:headers="headers"-->
            <!--:beforeUpload="beforeUpload"-->
            <!--:remove="imageRemove"-->
            <!--@preview="handlePreview"-->
            <!--@change="imageChange"-->
          <!--&gt;-->
            <!--<div v-if="fileList.length < 1">-->
              <!--<a-icon type="plus"/>-->
              <!--<div class="ant-upload-text">上 传</div>-->
            <!--</div>-->
          <!--</a-upload>-->
          <!--<a-modal :visible="previewVisible" :footer="null" @cancel="imageCancel">-->
            <!--<img alt="example" style="width: 100%" :src="previewImage"/>-->
          <!--</a-modal>-->
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="排版">
          <j-dict-select-tag v-decorator="['layout', {}]" :triggerChange="true" placeholder="请输入排版" dictCode="ads_layout"/>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="背景色">
          <a-input placeholder="请输入背景色" v-decorator="['bgColor', {}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="排序">
          <a-input v-decorator="[ 'sort', {}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="状态">
          <a-switch checkedChildren="启用" unCheckedChildren="停用" :checked="state == '0' ? true : false" @click="e => handelCheck(e)"/>
        </a-form-item>
		
      </a-form>
    </a-spin>
  </a-modal>
</template>

<script>
  import { httpAction,postAction} from '@/api/manage'
  import pick from 'lodash.pick'
  import moment from "moment"
  import Vue from 'vue'
  import {ACCESS_TOKEN} from "@/store/mutation-types"

  export default {
    name: "HomepageAdsModal",
    data () {
      return {
        title:"操作",
        visible: false,
        state: '0',
        model: {},
        labelCol: {
          xs: { span: 24 },
          sm: { span: 5 },
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 16 },
        },
        headers: {},
        previewVisible: false,
        previewImage: '',
        fileList: [],
        removeFileList: [],
        confirmLoading: false,
        form: this.$form.createForm(this),
        validatorRules:{
        },
        url: {
          add: "/com.ufak/homepageAds/add",
          edit: "/com.ufak/homepageAds/edit",
          fileUpload: window._CONFIG['domianURL'] + "/sys/common/upload",
          removeFile: window._CONFIG['domianURL'] + "/sys/common/remove",
        },
      }
    },
    created () {
      const token = Vue.ls.get(ACCESS_TOKEN);
      this.headers = {"X-Access-Token": token}
    },
    computed: {
      uploadAction: function () {
        return this.url.fileUpload;
      }
    },
    methods: {
      add () {
        this.edit({state:'0'});
      },
      edit (record) {
        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.visible = true;
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model,'type','headImg','layout','bgColor','sort'));

        });
        this.state = record.state;

        /** 图片渲染 **/
        if(this.model.imgUrl){
          let tmpUrls = this.model.imgUrl.split(',');
          let tmpIdx = 0;
          let baseUrl = window._CONFIG['domianURL'] + "/sys/common/view/";
          for(let i=0;i<tmpUrls.length;i++){
            let imgUrl = tmpUrls[i];
            tmpIdx--;
            this.fileList[i] = {
              uid: tmpIdx, //根据官方API文档，该值最好给个负数值，以免与内部对象冲突
              name: imgUrl.substring(imgUrl.lastIndexOf("/"),imgUrl.indexOf(".")),
              status: 'done',
              url: baseUrl + imgUrl,
              thumbUrl: baseUrl + imgUrl,
              isCommit: true, //图片是否已提交
            }
          }
        }else{
          this.fileList = [];
        }
        console.log("图片渲染fileList:",this.fileList);


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

            if (this.fileList.length == 0) {
              that.$message.warning("请上传商品图片！");
              that.confirmLoading = false;
              return;
            }
            console.log("this.fileList=", this.fileList);
            let imageUrls = "";
            for (let i = 0; i < this.fileList.length; i++) {
              let img = this.fileList[i];
              if (img.response) {
                imageUrls += img.response.message
              } else {
                imageUrls += img.url.substring(img.url.indexOf("/files"), img.url.length)
              }
            }
            console.log("图片路径：" + imageUrls);
            formData.imgUrl = imageUrls;
            formData.state = this.state;

            
            console.log(formData)
            httpAction(httpurl,formData,method).then((res)=>{
              if(res.success){
                // 提交成功后服务器删除移除的图片
                this.handleRemoveFile(this.removeFileList);
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
        let files = [];
        for (let i = 0; i < this.fileList.length; i++) {
          let img = this.fileList[i];
          if (!img.isCommit) {
            files.push(img.response.message);
          }
        }
        console.log("未提交的图片：",files);
        // 关闭成功后服务器删除未提交的图片
        this.handleRemoveFile(files);
        this.close()
      },
      handelCheck(check){
          if(check){
            this.state = '0';
          }else {
            this.state = '1';
          }
      },
      imageCancel() {
        this.previewVisible = false;
      },
      handlePreview(file) {
        this.previewImage = file.url || file.thumbUrl;
        this.previewVisible = true;
      },
      imageChange(info) {
        this.fileList = info.fileList;
        console.log("上传后：",this.fileList);
      },
      imageRemove(file) {
        if(file.url){
          let rmUrl = file.url.substring(file.url.indexOf("/files"), file.url.length);
          this.removeFileList.push(rmUrl);
        }else{
          if(file.response){
            this.removeFileList.push(file.response.message);
          }
        }

        console.log("删除图片路径：", this.removeFileList);
      },
      beforeUpload: function (file) {
        var fileType = file.type;
        if(fileType.indexOf('image')<0){
          this.$message.warning('请上传图片');
          return false;
        }
        const isLt2M = file.size / 1024 / 1024 < 0.5;
        if (!isLt2M) {
          this.$message.error('上传照片不要超过0.5MB!');
          return false;
        }
      },
      handleRemoveFile(files){
        postAction(this.url.removeFile, {filePaths:files});
      }

    }
  }
</script>

<style lang="less" scoped>

</style>