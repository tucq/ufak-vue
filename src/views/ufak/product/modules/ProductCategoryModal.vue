<template>
  <a-modal
    :title="title"
    :width="width"
    :visible="visible"
    :maskClosable="false"
    :confirmLoading="confirmLoading"
    @ok="handleOk"
    @cancel="handleCancel"
    :destroyOnClose="true"
    cancelText="关闭">
    <a-spin :spinning="confirmLoading">
      <a-form :form="form">

        <a-form-item label="父级节点" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-tree-select
            ref="treeSelect"
            placeholder="请选择父级节点"
            v-decorator="['pid', validatorRules.pid]"
            dict="t_product_category,name,id"
            pidField="pid"
            pidValue="0">
          </j-tree-select>
        </a-form-item>

        <a-form-item label="分类名称" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'name', validatorRules.name]" placeholder="请输入类型名称"></a-input>
        </a-form-item>

        <a-form-item label="分类图片" :labelCol="labelCol" :wrapperCol="wrapperCol">
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


      </a-form>
    </a-spin>
  </a-modal>
</template>

<script>

  import {httpAction, getAction, postAction} from '@/api/manage'
  import pick from 'lodash.pick'
  import JTreeSelect from '@/components/jeecg/JTreeSelect'
  import {duplicateCheck} from '@/api/api'
  import Vue from 'vue'
  import {ACCESS_TOKEN} from "@/store/mutation-types"

  export default {
    name: "ProductCategoryModal",
    components: {
      JTreeSelect
    },
    data() {
      return {
        form: this.$form.createForm(this),
        title: "操作",
        width: 800,
        visible: false,
        model: {},
        labelCol: {
          xs: {span: 24},
          sm: {span: 5},
        },
        wrapperCol: {
          xs: {span: 24},
          sm: {span: 16},
        },
        categoryId: '',
        headers: {},
        previewVisible: false,
        previewImage: '',
        fileList: [],
        removeFileList: [],
        confirmLoading: false,
        validatorRules: {
          code: {
            rules: [{
              required: true, message: '请输入类型编码!'
            }, {
              validator: this.validateMyCode
            }]
          },
          pid: {},
          name: {
            rules: [{required: true, message: '请输入类型名称!'}, {validator: this.validateName}]
          },
        },
        url: {
          add: "/product/category/add",
          edit: "/product/category/edit",
          checkCode: "/product/category/checkCode",
          fileUpload: window._CONFIG['domianURL'] + "/sys/common/upload",
          removeFile: window._CONFIG['domianURL'] + "/sys/common/remove",
        },
        expandedRowKeys: [],
        pidField: "pid"

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
        this.edit({});
      },
      edit(record) {
        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.visible = true;
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model, 'pid', 'name', 'code'))
        })

        this.categoryId = record.id;

        /** 图片渲染 **/
        if (this.model.image) {
          let tmpUrls = this.model.image.split(',');
          let tmpIdx = 0;
          let baseUrl = window._CONFIG['domianURL'] + "/sys/common/view/";
          for (let i = 0; i < tmpUrls.length; i++) {
            let imgUrl = tmpUrls[i];
            tmpIdx--;
            this.fileList[i] = {
              uid: tmpIdx, //根据官方API文档，该值最好给个负数值，以免与内部对象冲突
              name: imgUrl.substring(imgUrl.lastIndexOf("/"), imgUrl.indexOf(".")),
              status: 'done',
              url: baseUrl + imgUrl,
              thumbUrl: baseUrl + imgUrl,
              isCommit: true, //图片是否已提交
            }
          }
        } else {
          this.fileList = [];
        }
        console.log("图片渲染fileList:", this.fileList);

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


            // if ((formData.pid && formData.pid != 0) && this.fileList.length == 0) {
            //   that.$message.warning("请上传商品图片！");
            //   that.confirmLoading = false;
            //   return;
            // }
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
            console.log("商品图片路径：" + imageUrls);

            formData.image = imageUrls;

            console.log("表单提交数据", formData)
            httpAction(httpurl, formData, method).then((res) => {
              if (res.success) {
                // 提交成功后服务器删除移除的图片
                this.handleRemoveFile(this.removeFileList);

                that.$message.success(res.message);
                that.submitSuccess(formData)
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
        console.log("未提交的图片：", files);
        // 关闭成功后服务器删除未提交的图片
        this.handleRemoveFile(files);

        this.close()
      },
      popupCallback(row) {
        this.form.setFieldsValue(pick(row, 'pid', 'name', 'code'))
      },
      submitSuccess(formData) {
        if (!formData.id) {
          let treeData = this.$refs.treeSelect.getCurrTreeData()
          this.expandedRowKeys = []
          this.getExpandKeysByPid(formData[this.pidField], treeData, treeData)
          this.$emit('ok', formData, this.expandedRowKeys.reverse());
        } else {
          this.$emit('ok', formData);
        }
      },
      getExpandKeysByPid(pid, arr, all) {
        if (pid && arr && arr.length > 0) {
          for (let i = 0; i < arr.length; i++) {
            if (arr[i].key == pid) {
              this.expandedRowKeys.push(arr[i].key)
              this.getExpandKeysByPid(arr[i]['parentId'], all, all)
            } else {
              this.getExpandKeysByPid(pid, arr[i].children, all)
            }
          }
        }
      },
      validateMyCode(rule, value, callback) {
        let params = {
          pid: this.form.getFieldValue('pid'),
          code: value
        }
        getAction(this.url.checkCode, params).then((res) => {
          if (res.success) {
            callback()
          } else {
            callback(res.message)
          }
        })
      },
      validateName(rule, value, callback) {
        var params = {
          tableName: 't_product_category',
          fieldName: 'name',
          fieldVal: value,
          dataId: this.categoryId
        };
        duplicateCheck(params).then((res) => {
          if (res.success) {
            callback()
          } else {
            callback("类型名称已存在!")
          }
        })
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
        console.log("上传后：", this.fileList);
      },
      imageRemove(file) {
        if (file.url) {
          let rmUrl = file.url.substring(file.url.indexOf("/files"), file.url.length);
          this.removeFileList.push(rmUrl);
        } else {
          if (file.response) {
            this.removeFileList.push(file.response.message);
          }
        }

        console.log("删除图片路径：", this.removeFileList);
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
        postAction(this.url.removeFile, {filePaths: files}).then((res) => {

        });
      }


    }
  }
</script>