<template>
  <a-modal
    :title="title"
    :width="900"
    style=" top: 0px;"
    :visible="visible"
    :maskClosable="false"
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
              label="商品名称">
              <a-input placeholder="请输入商品名称" v-decorator="['name', validatorRules.name]" maxlength="200"/>
            </a-form-item>
          </a-col>
          <a-col :span="12">
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="商品描述">
              <a-input placeholder="请输入商品描述" v-decorator="['description', validatorRules.description]" maxlength="200"/>
            </a-form-item>
          </a-col>
        </a-row>

        <a-row :gutter="20">
          <a-col :span="12">
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="商品分类">
                <j-tree-select
                  ref="treeSelect"
                  placeholder="请选择商品分类"
                  v-decorator="['productType', validatorRules.productType]"
                  dict="t_product_category,name,id"
                  pidField="pid"
                  pidValue="0">
                </j-tree-select>
            </a-form-item>
          </a-col>
          <a-col :span="12">
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="上下架">
              <j-dict-select-tag v-decorator="['state', validatorRules.state]" :triggerChange="true" placeholder="请选择上下架" dictCode="product_state"/>
            </a-form-item>
          </a-col>
        </a-row>

        <a-row :gutter="20">
          <a-col :span="12">
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="销量">
              <a-row :gutter="8">
                <a-col :span="12">
                  <a-input v-decorator="[ 'salesVolume', {}]" disabled/>
                </a-col>
                <a-col :span="12">
                  <a-form-item style="margin-bottom:0px">
                    <a-input placeholder="虚似销量" v-decorator="[ 'virtualVolume', validatorRules.virtualVolume]" />
                  </a-form-item>
                </a-col>
              </a-row>
            </a-form-item>
          </a-col>
          <a-col :span="12">
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="运费">
              <a-input placeholder="免运费请输0" v-decorator="['freightAmount', validatorRules.freightAmount]" maxlength="200"/>
            </a-form-item>
          </a-col>
        </a-row>

        <a-row :gutter="20">
          <a-col :span="12">
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="退/换/维">
              <a-row :gutter="8">
                <a-col :span="8">
                  <a-form-item style="margin-bottom:0px">
                    <a-input placeholder="退货有效天数" v-decorator="[ 'refundValidity', validatorRules.refundValidity]" />
                  </a-form-item>
                </a-col>
                <a-col :span="8">
                  <a-form-item style="margin-bottom:0px">
                    <a-input placeholder="换货有效天数" v-decorator="[ 'barterValidity', validatorRules.barterValidity]" />
                  </a-form-item>
                </a-col>
                <a-col :span="8">
                  <a-form-item style="margin-bottom:0px">
                    <a-input placeholder="维修有效天数" v-decorator="[ 'repairValidity', validatorRules.repairValidity]" />
                  </a-form-item>
                </a-col>
              </a-row>
            </a-form-item>
          </a-col>
          <a-col :span="12">
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="推荐理由">
              <a-input placeholder="多个理由用#号分隔" v-decorator="['keyPoint', {}]" maxlength="200"/>
            </a-form-item>
          </a-col>
        </a-row>

        <a-row :gutter="20">
          <a-col :span="12">
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="服务">
              <a-select
                mode="multiple"
                placeholder="请选择服务"
                v-model="serviceValue"
                @change="handleChangeService"
              >
                <a-select-option v-for="(item, key) in  serviceList" :key="key" :value="item.text">
                  <span style="display: inline-block;width: 100%" :title=" item.text || item.label ">
                    {{ item.text || item.label }}
                  </span>
                </a-select-option>
              </a-select>
            </a-form-item>
          </a-col>
        </a-row>

        <a-row :gutter="20" style="margin-left: 10px;margin-right: 10px">
          <a-tabs defaultActiveKey="1">
            <a-tab-pane key="1">
              <span slot="tab">
                <a-icon type="picture"/>商品图片
              </span>
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
                  <div class="ant-upload-text">上传主图</div>
                </div>
              </a-upload>
              <a-modal :visible="previewVisible" :footer="null" @cancel="imageCancel">
                <img alt="example" style="width: 100%" :src="previewImage"/>
              </a-modal>

            </a-tab-pane>
            <a-tab-pane key="2">
              <span slot="tab">
                <a-icon type="file-jpg"/>详情图片
              </span>
              <product-detail-images ref="productDetailImages" :headers="headers" @beforeUpload="beforeUpload" @imageRemove="imageRemove"
                    :parentDetailImages="detailImages"  @changeDetailImages="changeDetailImages"/>
            </a-tab-pane>
            <a-tab-pane key="3">
              <span slot="tab">
                <a-icon type="setting"/>商品规格
              </span>
              <product-specs ref="productSpecs" @dataOne="handelDataOne" @dataTwo="handelDataTwo"
                             @specsTitleOne="handelSpecsTitleOne" @specsTitleTwo="handelSpecsTitleTwo"
                             @removeProductSpecsList="handelRemoveSpecsList" :productSpecsList="productSpecsList"></product-specs>
            </a-tab-pane>


          </a-tabs>
        </a-row>


      </a-form>
    </a-spin>
  </a-modal>
</template>

<script>
    import {httpAction, postAction,getAction} from '@/api/manage'
    import {ajaxGetDictItems} from '@/api/api'
    import JTreeSelect from '@/components/jeecg/JTreeSelect'
    import pick from 'lodash.pick'
    import moment from "moment"
    import Vue from 'vue'
    import {ACCESS_TOKEN} from "@/store/mutation-types"
    import ProductDetailImages from "./ProductDetailImages";
    import ProductSpecs from "./ProductSpecs";
//    import ProductPrice from "./ProductPrice";
    import AInput from "ant-design-vue/es/input/Input";

    export default {
        name: "ProductInfoModal",
        components: {
          AInput, ProductDetailImages,
            ProductSpecs,
//            ProductPrice,
            JTreeSelect,
        },
        data() {
            return {
                title: "操作",
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
                specsTitleOne:'',
                specsTitleTwo:'',
                dataOne: [],
                dataTwo: [],
                confirmLoading: false,
                form: this.$form.createForm(this),
                validatorRules: {},
                headers: {},
                previewVisible: false,
                previewImage: '',
                fileList: [],
                removeFileList: [],
                productSpecsList: [],
                detailImages: '',
                removeProductSpecsList: [],
                serviceList: [],
                serviceValue: [],
                validatorRules: {
                  name: { rules: [{ required: true, message: '请输入商品名称' }] },
                  description: { rules: [{ required: true, message: '请输入商品描述' }] },
                  productType: { rules: [{ required: true, message: '请选择商品分类' }] },
                  state: { rules: [{ required: true, message: '请选择上下架' }] },
                  virtualVolume: { rules: [{ validator: this.validatorVirtualVolume }] },
                  freightAmount: { rules: [{ validator: this.validatorFreightAmount }] },
                  refundValidity: { rules: [{ validator: this.validatorVirtualVolume }] },
                  barterValidity: { rules: [{ validator: this.validatorVirtualVolume }] },
                  repairValidity: { rules: [{ validator: this.validatorVirtualVolume }] },
                },
                url: {
                    add: "/product/info/add",
                    edit: "/product/info/edit",
                    getProductSpecsList: "/product/specs/list",
                    fileUpload: window._CONFIG['domianURL'] + "/sys/common/upload",
                    removeFile: window._CONFIG['domianURL'] + "/sys/common/remove",
                },
            }
        },
        created() {
            const token = Vue.ls.get(ACCESS_TOKEN);
            this.headers = {"X-Access-Token": token}
            this.initDictData();
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
                    this.form.setFieldsValue(pick(this.model, 'name', 'description', 'productType', 'salesVolume', 'service', 'state','virtualVolume','keyPoint','freightAmount','refundValidity','barterValidity','repairValidity'))
                    //时间格式化
                });
                if(record.service){
                    this.serviceValue = record.service.split(',');
                }else{
                  this.serviceValue = [];
                }

                /** 图片渲染 **/
                this.fileList = [];
                if(this.model.image){
                    let tmpUrls = this.model.image.substring(0,this.model.image.length-1).split(',');
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
                this.detailImages = this.model.detailImages;

                // 查询规格数据
                if(record.id){
                    this.initProductSpecsList(record.id);
                }else{
                    this.initProductSpecsList(null);
                }

            },
            close() {
                this.$emit('close');
                this.visible = false;
            },
            handleOk() {
                const that = this;
                if (this.fileList.length == 0) {
                    that.$message.warning("请上传商品图片！");
                    return;
                }
                if(!this.specsTitleOne || this.dataOne.length == 0){
                    that.$message.warning("请完善商品一级规格内容！");
                    return
                }

                /****** 商品图片 *******/
                console.log("this.fileList=", this.fileList);
                let imageUrls = "";
                for (let i = 0; i < this.fileList.length; i++) {
                    let img = this.fileList[i];
                    if (img.response) {
                        imageUrls += img.response.message + ","
                    } else {
                        imageUrls += img.url.substring(img.url.indexOf("/files"), img.url.length) + ","
                    }
                }
                console.log("商品图片路径：" + imageUrls);

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
                        formData.image = imageUrls;

                        /******** 商品规格 *********/
                        this.setProductSpecsList();
                        formData.productSpecsList = this.productSpecsList;
                        formData.removeProductSpecsList = this.removeProductSpecsList;
                        /******** 商品规格 *********/

                        formData.detailImages = this.detailImages;//明细图片

                        formData.service = this.serviceValue.join(','); // 服务

                        console.log(formData)
                        httpAction(httpurl, formData, method).then((res) => {
                            if (res.success) {
                                // 提交成功后服务器删除移除的图片
                                this.handleRemoveFile(this.removeFileList);

                                that.$message.success(res.message);
                                that.$emit('ok',res.result);
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
                // 关闭成功后服务器删除未提交的图片
                this.handleRemoveFile(files);

                if(this.$refs.productDetailImages){ // tab标签可能没有加载
                  this.$refs.productDetailImages.clearRemoveFile();//删除没用的图片
                }

                this.close()
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
                console.log(this.fileList);
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
            beforeUpload: function (file) {
//                const isJPG = file.type === 'image/jpeg';
//                if (!isJPG) {
//                    this.$message.error('请上传jpg格式图片!');
//                }
                const isLt2M = file.size / 1024 / 1024 < 2;
                if (!isLt2M) {
                    this.$message.error('上传照片不要超过2MB!');
                }
//                return isJPG && isLt2M;
                return isLt2M;
            },
            handleRemoveFile(files){
                postAction(this.url.removeFile, {filePaths:files}).then((res) => {

                });
            },
            handelDataOne(obj){
              this.dataOne = obj;
            },
            handelDataTwo(obj){
              this.dataTwo = obj;
            },
            handelSpecsTitleOne(value){
              this.specsTitleOne = value;
            },
            handelSpecsTitleTwo(value){
              this.specsTitleTwo = value;
            },
            handelRemoveSpecsList(obj){
                this.removeProductSpecsList = obj;
            },
            initProductSpecsList(productId){
                if(productId == null){
                    this.productSpecsList = [];
                    return;
                }
                let params = {
                    productId : productId,
                    orderBy:'sort',
                    pageNo: 1,
                    pageSize: 2147483647
                };
                getAction(this.url.getProductSpecsList, params).then((res)=>{
                    if(res.success){
                        this.productSpecsList = res.result.records;
                    }
                });
            },
            setProductSpecsList(){
                let productSpecsList = [];
                if(this.specsTitleOne.specsTitle){
                    productSpecsList.push(this.specsTitleOne);
                }
                if(this.dataOne.length > 0){
                    for(let i=0;i<this.dataOne.length;i++){
                        this.dataOne[i].pid = 'pid';//给个无用值,不然有些浏览器会自动赋值0
                        productSpecsList.push(this.dataOne[i]);
                    }
                }

                if(this.specsTitleTwo.specsTitle){
                    productSpecsList.push(this.specsTitleTwo);
                }
                if(this.dataTwo.length > 0){
                    for(let i=0;i<this.dataTwo.length;i++){
                      this.dataTwo[i].pid = 'pid';//给个无用值,不然有些浏览器会自动赋值0
                        productSpecsList.push(this.dataTwo[i]);
                    }
                }

                this.productSpecsList = productSpecsList;
            },
            changeDetailImages(detailImages){
                this.detailImages = detailImages;
                console.log("changeDetailImages: ",detailImages);
            },
            initDictData() {
              ajaxGetDictItems('product_service', null).then((res) => {
                if (res.success) {
                  this.serviceList = res.result;
                }
              })
            },
            handleChangeService(val){
              this.serviceValue = val;
            },
            validatorVirtualVolume(rule,value,callback){
              if(value){
                let s = /^[+]{0,1}(\d+)$/;
                if(!s.test(value)){
                  callback('请输入正整数');
                }else{
                  callback();
                }
              }else{
                callback();
              }
            },
            validatorFreightAmount(rule,value,callback){
              if(value){
                let s = /^\d+(\.\d+)?$/;
                if(!s.test(value)){
                  callback('请输入合法数字');
                }else{
                  callback();
                }
              }else{
                callback();
              }
            },


        }
    }
</script>

<style lang="less" scoped>
  /*you can make up upload button and sample style by using stylesheets*/
  .ant-upload-select-picture-card i {
    font-size: 16px;
    color: #999;
  }

  .ant-upload-select-picture-card .ant-upload-text {
    margin-top: 8px;
    color: #666;
  }


</style>