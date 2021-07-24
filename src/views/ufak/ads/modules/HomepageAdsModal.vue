<template>
  <a-modal
    :title="title"
    :width="800"
    :visible="visible"
    :maskClosable="false"
    :confirmLoading="confirmLoading"
    @ok="handleOk"
    @cancel="handleCancel"
    cancelText="关闭">

    <a-spin :spinning="confirmLoading">
      <a-form :form="form">
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="广告名称">
          <a-input placeholder="请输入请输入广告名称" v-decorator="['adsName', validatorRules.adsName]"/>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="广告类型">
          <j-dict-select-tag v-decorator="['type', validatorRules.type]" :triggerChange="true" placeholder="请输入广告类型"
                             dictCode="ads_type"/>
        </a-form-item>

        <a-row>
          <a-col :span="13" style="padding-left: 90px;">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="广告图片">
              <a-upload
                :action="uploadAction"
                listType="picture-card"
                :fileList="fileList"
                :headers="headers"
                :beforeUpload="beforeUpload"
                :remove="e => imageRemove(e,1)"
                @preview="e => handlePreview(e,1)"
                @change="e => imageChange(e,1)"
              >
                <div v-if="fileList.length < 1">
                  <a-icon type="plus"/>
                  <div class="ant-upload-text">上 传</div>
                </div>
              </a-upload>
              <a-modal :visible="previewVisible" :footer="null" @cancel="() => imageCancel(1)">
                <img alt="example" style="width: 100%" :src="previewImage"/>
              </a-modal>
            </a-form-item>
          </a-col>
          <a-col :span="11">
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="详情头部">
              <!--<a-input placeholder="请输入详情头部" v-decorator="['headImg', {}]" />-->
              <a-upload
                :action="uploadAction"
                listType="picture-card"
                :fileList="fileList2"
                :headers="headers"
                :beforeUpload="beforeUpload"
                :remove="e => imageRemove(e,2)"
                @preview="e => handlePreview(e,2)"
                @change="e => imageChange(e,2)"
              >
                <div v-if="fileList2.length < 1">
                  <a-icon type="plus"/>
                  <div class="ant-upload-text">上 传</div>
                </div>
              </a-upload>
              <a-modal :visible="previewVisible2" :footer="null" @cancel="() => imageCancel(2)">
                <img alt="example" style="width: 100%" :src="previewImage2"/>
              </a-modal>
            </a-form-item>
          </a-col>
        </a-row>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="背景色">
          <a-input placeholder="请输入背景色" v-decorator="['bgColor', {}]"/>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="排序">
          <a-input v-decorator="[ 'sort',validatorRules.sort]" placeholder="请输入排序"/>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="是否列表">
          <a-select
            placeholder="请输入是否列表"
            v-decorator="['isList', {}]"
          >
            <a-select-option value="1">否</a-select-option>
            <a-select-option value="0">是</a-select-option>
          </a-select>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="页面路径">
          <a-input placeholder="请输入页面跳转路径" v-decorator="['pagePath', {}]"/>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="状态">
          <a-switch checkedChildren="启用" unCheckedChildren="停用" :checked="state == '0' ? true : false"
                    @click="e => handelCheck(e)"/>
        </a-form-item>

      </a-form>
    </a-spin>
  </a-modal>
</template>

<script>
  import {httpAction, postAction} from '@/api/manage'
  import pick from 'lodash.pick'
  import moment from "moment"
  import Vue from 'vue'
  import {ACCESS_TOKEN} from "@/store/mutation-types"
  import ARow from "ant-design-vue/es/grid/Row";
  import ACol from "ant-design-vue/es/grid/Col";

  export default {
    components: {ACol, ARow}, name: "HomepageAdsModal",
    data() {
      return {
        title: "操作",
        visible: false,
        state: '0',
        model: {},
        labelCol: {
          xs: {span: 24},
          sm: {span: 5},
        },
        wrapperCol: {
          xs: {span: 24},
          sm: {span: 16},
        },
        headers: {},
        opt: '',
        previewVisible: false,
        previewVisible2: false,
        previewImage: '',
        previewImage2: '',
        fileList: [],
        fileList2: [],
        removeFileList: [],
        removeFileList2: [],
        confirmLoading: false,
        form: this.$form.createForm(this),
        validatorRules: {
          adsName: {rules: [{required: true, message: '请输入广告名称!'}]},
          type: {rules: [{required: true, message: '请输入广告类型!'}]},
          sort: {rules: [{required: true, message: '请输入排序!'}]},
        },
        url: {
          add: "/homepageAds/add",
          edit: "/homepageAds/edit",
          fileUpload: window._CONFIG['domianURL'] + "/sys/common/upload",
          removeFile: window._CONFIG['domianURL'] + "/sys/common/remove",
        },
      }
    },
    created() {
      const token = Vue.ls.get(ACCESS_TOKEN);
      this.headers = {"X-Access-Token": token}
    },
    computed: {
      uploadAction: function () {
        return this.url.fileUpload;
      }
    },
    methods: {
      add() {
        this.edit({state: '0', isList: '0'});
        this.opt = 'add';
      },
      edit(record) {
        this.opt = 'edit';
        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.visible = true;
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model, 'adsName', 'type', 'bgColor', 'sort', 'isList', 'pagePath'));

        });
        this.state = record.state;

        /** 图片渲染 **/
        let url = this.model.imgUrl;
        if (url) {
          this.renderHtml(url, 1);
        } else {
          this.fileList = [];
        }
        let url2 = this.model.headImg;
        if (url2) {
          this.renderHtml(url2, 2);
        } else {
          this.fileList2 = [];
        }

      },
      renderHtml(url, flag) {
        let tmpUrls = url.split(',');
        let tmpIdx = 0;
        let baseUrl = window._CONFIG['domianURL'] + "/sys/common/view/";
        for (let i = 0; i < tmpUrls.length; i++) {
          let imgUrl = tmpUrls[i];
          tmpIdx--;
          let fileObj = {
            uid: tmpIdx, //根据官方API文档，该值最好给个负数值，以免与内部对象冲突
            name: imgUrl.substring(imgUrl.lastIndexOf("/"), imgUrl.indexOf(".")),
            status: 'done',
            url: baseUrl + imgUrl,
            thumbUrl: baseUrl + imgUrl,
            isCommit: true, //图片是否已提交
          };

          if (flag === 1) {
            this.fileList[i] = fileObj;
          } else {
            this.fileList2[i] = fileObj;
          }
        }
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
            formData.state = this.state;

            if (this.fileList.length == 0) {
              that.$message.warning("请上传广告图片！");
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
                imageUrls += img.url.substring(img.url.indexOf("/files") + 1, img.url.length)
              }
            }
            formData.imgUrl = imageUrls;

            let imageUrls2 = "";
            for (let i = 0; i < this.fileList2.length; i++) {
              let img = this.fileList2[i];
              if (img.response) {
                imageUrls2 += img.response.message
              } else {
                imageUrls2 += img.url.substring(img.url.indexOf("/files") + 1, img.url.length)
              }
            }
            formData.headImg = imageUrls2;

            console.log(formData)
            httpAction(httpurl, formData, method).then((res) => {
              if (res.success) {
                // 提交成功后服务器删除移除的图片
                this.handleRemoveFile(this.removeFileList);
                this.handleRemoveFile(this.removeFileList2);
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
        let files = [];
        for (let i = 0; i < this.fileList.length; i++) {
          let img = this.fileList[i];
          if (!img.isCommit) {
            files.push(img.response.message);
          }
        }
        for (let i = 0; i < this.fileList2.length; i++) {
          let img = this.fileList2[i];
          if (!img.isCommit) {
            files.push(img.response.message);
          }
        }
        console.log("未提交的图片：", files);
        // 关闭成功后服务器删除未提交的图片
        this.handleRemoveFile(files);

        if (this.opt == 'add') {
          this.handleRemoveFile(this.removeFileList);
          this.handleRemoveFile(this.removeFileList2);
        }

        this.close()
      },
      handelCheck(check) {
        if (check) {
          this.state = '0';
        } else {
          this.state = '1';
        }
      },
      imageCancel(flag) {
        if (flag === 1) {
          this.previewVisible = false;
        } else {
          this.previewVisible2 = false;
        }
      },
      handlePreview(file, flag) {
        if (flag === 1) {
          this.previewImage = file.url || file.thumbUrl;
          this.previewVisible = true;
        } else {
          this.previewImage2 = file.url || file.thumbUrl;
          this.previewVisible2 = true;
        }
      },
      imageChange(info, flag) {
        if (flag === 1) {
          this.fileList = info.fileList;
        } else {
          this.fileList2 = info.fileList;
        }
        console.log("上传后：", this.fileList);
      },
      imageRemove(file, flag) {
        if (file.url) {
          let rmUrl = file.url.substring(file.url.indexOf("/files"), file.url.length);
          if (flag === 1) {
            this.removeFileList.push(rmUrl);
          } else {
            this.removeFileList2.push(rmUrl);
          }
        } else {
          if (file.response) {
            if (flag === 1) {
              this.removeFileList.push(file.response.message);
            } else {
              this.removeFileList2.push(file.response.message);
            }

          }
        }
      },
      beforeUpload: function (file) {
        var fileType = file.type;
        if (fileType.indexOf('image') < 0) {
          this.$message.warning('请上传图片');
          return false;
        }
        const isLt2M = file.size / 1024 / 1024 < 0.5;
        if (!isLt2M) {
          this.$message.error('上传照片不要超过0.5MB!');
          return false;
        }
      },
      handleRemoveFile(files) {
        postAction(this.url.removeFile, {filePaths: files});
      }

    }
  }
</script>

<style lang="less" scoped>

</style>