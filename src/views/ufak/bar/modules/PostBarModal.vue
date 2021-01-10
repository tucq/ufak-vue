<template>
  <a-modal
    :title="title"
    :width="800"
    :visible="visible"
    style="top:0px;"
    :confirmLoading="confirmLoading"
    @ok="handleOk"
    @cancel="handleCancel"
    cancelText="关闭">

    <a-spin :spinning="confirmLoading">
      <a-form :form="form">
        <a-row :gutter="20">
          <a-col :span="12">
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="分类">
              <a-select
                placeholder="请输入分类"
                v-decorator="['categoryId', {}]"
              >
                <a-select-option :value="option.id" v-for="option in categoryList">
                  {{ option.itemText }}
                </a-select-option>
              </a-select>
            </a-form-item>
          </a-col>
          <a-col :span="12">
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="置顶顺序">
              <a-input v-decorator="[ 'top', {}]" />
            </a-form-item>
          </a-col>
        </a-row>
        <a-row :gutter="20">
          <a-col :span="24">
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="标题">
              <a-input placeholder="请输入标题" v-decorator="['title', {}]" />
            </a-form-item>
          </a-col>
        </a-row>

        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="内容">
          <a-textarea  rows="10" placeholder="请输入内容" v-decorator="['content', {}]" />
        </a-form-item>

        <a-row :gutter="20" style="margin-left: 30px;margin-right: 10px">
          <a-col :span="24">
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
              <div v-if="fileList.length < 10">
                <a-icon type="plus"/>
                <div class="ant-upload-text">上传</div>
              </div>
            </a-upload>
            <a-modal :visible="previewVisible" :footer="null" @cancel="imageCancel">
              <img alt="example" style="width: 100%" :src="previewImage"/>
            </a-modal>
          </a-col>
        </a-row>


        <a-row :gutter="20">
          <a-col :span="12">
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="用户ID">
              <a-input placeholder="请输入用户id" v-decorator="['userId', {}]" disabled />
            </a-form-item>
          </a-col>
          <a-col :span="12">
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="真实姓名">
              <a-input placeholder="请输入真实姓名" v-decorator="['realname', {}]" disabled />
            </a-form-item>
          </a-col>
        </a-row>

        <a-row :gutter="20">
          <a-col :span="12">
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="头像">
              <a-input placeholder="请输入头像" v-decorator="['avatar', {}]" disabled />
            </a-form-item>
          </a-col>
          <a-col :span="12">
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="分享数量">
              <a-input v-decorator="[ 'shareNum', {}]" />
            </a-form-item>
          </a-col>
        </a-row>

        <a-row :gutter="20">
          <a-col :span="12">
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="点赞数量">
              <a-input v-decorator="[ 'likesNum', {}]" />
            </a-form-item>
          </a-col>
          <a-col :span="12">
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="状态">
              <a-select
                placeholder="请输入状态"
                v-decorator="['state', {}]"
              >
                <a-select-option value="0">显示</a-select-option>
                <a-select-option value="1">禁言</a-select-option>
              </a-select>
            </a-form-item>
          </a-col>
        </a-row>



      </a-form>
    </a-spin>
  </a-modal>
</template>

<script>
  import { httpAction,postAction,getAction } from '@/api/manage'
  import pick from 'lodash.pick'
  import moment from "moment"
  import Vue from 'vue'
  import {ACCESS_TOKEN} from "@/store/mutation-types"
  import ATextarea from "ant-design-vue/es/input/TextArea";

  export default {
    components: {ATextarea}, name: "PostBarModal",
    data () {
      return {
        headers: {},
        categoryList:[],
        fileList: [],
        removeFileList: [],
        previewImage: '',
        previewVisible: false,
        title:"操作",
        visible: false,
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
          add: "/postBar/add",
          edit: "/postBar/edit",
          categoryList: "/sys/dict/item/bar_category",
          fileUpload: window._CONFIG['domianURL'] + "/sys/common/byType/upload?imgType=bar",
          removeFile: window._CONFIG['domianURL'] + "/sys/common/remove",
        },
      }
    },
    computed: {
      uploadAction: function () {
        return this.url.fileUpload;
      }
    },
    created () {
      const token = Vue.ls.get(ACCESS_TOKEN);
      this.headers = {"X-Access-Token": token}
      this.getCategoryList();
    },
    methods: {
      getCategoryList(){
        getAction(this.url.categoryList).then((res)=>{
          if(res.success){
            this.categoryList = res.result;
          }
        });
      },
      beforeUpload: function (file) {
        const isJPG = file.type === 'image/jpeg';
        if (!isJPG) {
          this.$message.error('请上传jpg格式图片!');
        }
//        const isLt2M = file.size / 1024 / 1024 < 2;
//        if (!isLt2M) {
//          this.$message.warn('上传照片不要超过2MB!');
//        }
        return isJPG;
      },
      imageRemove(file) {
        if(file.url){
          let rmUrl = file.url.substring(file.url.indexOf("/files"), file.url.length);
          this.removeFileList.push(rmUrl);
        }else{
          this.removeFileList.push(file.response.message);
        }

        console.log("删除图片路径：", this.removeFileList);
      },
      handlePreview(file) {
        this.previewImage = file.url || file.thumbUrl;
        this.previewVisible = true;
      },
      imageChange(info) {
        this.fileList = info.fileList;
        console.log(this.fileList);
      },
      imageCancel() {
        this.previewVisible = false;
      },
      handleRemoveFile(files){
        postAction(this.url.removeFile, {filePaths:files}).then((res) => {

        });
      },
      /*******图片*******/

      add () {
        this.edit({"userId":"1347172483152826370","realname":"福安康 185 1819 9181","avatar":"https://thirdwx.qlogo.cn/mmopen/vi_32/EtnjMAJFZcX6vzbxFibRNsQI9RLHVibyqcfHCFg5BVgMI3CKPN3x1BsTmTFP9mibRTW00Myqib6XycHgW7zYchAdYQ/132","top":99});
      },
      edit (record) {
        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.visible = true;
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model,'categoryId','title','content','images','userId','realname','avatar','shareNum','likesNum','state','top'))
        });
        /** 图片渲染 **/
        this.fileList = [];
        if(this.model.images){
          let tmpUrls = this.model.images.substring(0,this.model.images.length-1).split(',');
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
        }
        /** 明细图片渲染 **/

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

            let imageUrls = "";
            for (let i = 0; i < this.fileList.length; i++) {
              let img = this.fileList[i];
              if (img.response) {
                imageUrls += img.response.message + ","
              } else {
                imageUrls += img.url.substring(img.url.indexOf("/files"), img.url.length) + ","
              }
            }
            formData.images = imageUrls;

            console.log(formData)
            httpAction(httpurl,formData,method).then((res)=>{
              if(res.success){
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
        // 关闭成功后服务器删除未提交的图片
        this.handleRemoveFile(files);
        this.close()
      },


    }
  }
</script>

<style lang="less" scoped>

</style>