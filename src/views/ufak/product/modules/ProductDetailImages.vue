<template>
  <div>
      <a-upload
        :action="uploadAction"
        listType="picture-card"
        :fileList="detailFileList"
        :headers="headers"
        :beforeUpload="beforeUpload"
        :remove="imageRemove"
        @preview="handlePreview"
        @change="imageChange"
      >
      <div>
        <a-icon type="plus"/>
        <div class="ant-upload-text">上传主图</div>
      </div>
    </a-upload>
    <a-modal :visible="previewVisible" :footer="null" @cancel="imageCancel">
      <img alt="example" style="width: 100%" :src="previewImage"/>
    </a-modal>
  </div>
</template>


<script>

  import {postAction} from '@/api/manage'

  export default {
    name: "ProductDetailImages",
    props: {
      headers: Object,
      parentDetailImages: String,
    },
    data() {
      return {
        previewVisible: false,
        previewImage: '',
        detailFileList: [],
        detailImages: '',
        url: {
          fileUpload: window._CONFIG['domianURL'] + "/sys/common/upload",
          removeFile: window._CONFIG['domianURL'] + "/sys/common/remove",
        },
      }
    },
    watch: {
      // 'parentDetailImages': function () {
      //     this.loadImage(this.parentDetailImages);
      // },
      'detailImages': function () {
          this.$emit('changeDetailImages', this.detailImages);
      },
    },
    created() {
      this.loadImage(this.parentDetailImages);
    },
    computed: {
      uploadAction: function () {
        return this.url.fileUpload;
      }
    },
    methods: {
      loadImage(parentDetailImages){
          console.log("加载明细图片",parentDetailImages);
        /** 图片渲染 **/
        this.detailFileList = [];
        if(parentDetailImages){
          this.detailImages = parentDetailImages;
          let tmpUrls = parentDetailImages.substring(0,parentDetailImages.length-1).split(',');
          let tmpIdx = 0;
          let baseUrl = window._CONFIG['domianURL'] + "/sys/common/view/";
//          let baseUrl = "";
          for(let i=0;i<tmpUrls.length;i++){
            let imgUrl = tmpUrls[i];
            tmpIdx--;
            this.detailFileList[i] = {
              uid: tmpIdx, //根据官方API文档，该值最好给个负数值，以免与内部对象冲突
              name: imgUrl.substring(imgUrl.lastIndexOf("/"),imgUrl.indexOf(".")),
              status: 'done',
              url: baseUrl + imgUrl,
              thumbUrl: baseUrl + imgUrl,
              isCommit: true, //图片是否已提交
            }
          }
        }
      },
      beforeUpload(file){
          this.$emit('beforeUpload', file);
      },
      handlePreview(file) {
        this.previewImage = file.url || file.thumbUrl;
        this.previewVisible = true;
      },
      imageRemove(file) {
        console.log("明细页面imageRemove",file);
        let rmUrl = '';
        if(file.url){
            rmUrl = file.url.substring(file.url.indexOf("files"), file.url.length)+",";
        }else{
            rmUrl = file.response.message + ",";
        }
        let reg = new RegExp(rmUrl,"g");
        this.detailImages = this.detailImages.replace(reg,'');

        this.$emit('imageRemove', file);
      },
      imageChange(info) {
        this.detailFileList = info.fileList;
        if (info.file.status === 'done') {
          let imageUrls = "";
          for (let i = 0; i < this.detailFileList.length; i++) {
            let img = this.detailFileList[i];
            if (img.response) {
              imageUrls += img.response.message + ","
            } else {
              imageUrls += img.url.substring(img.url.indexOf("/files"), img.url.length) + ","
            }
          }
          this.detailImages = imageUrls;
          // this.$emit('changeDetailImages', imageUrls);
          console.log("明细图选择后的地址",this.parentDetailImages);
        }

      },
      imageCancel() {
        this.previewVisible = false;
      },
      clearRemoveFile(){
        // 未提交的商品明细图
        let files = [];
        for (let i = 0; i < this.detailFileList.length; i++) {
          let img = this.detailFileList[i];
          if (!img.isCommit) {
            files.push(img.response.message);
          }
        }

        this.handleRemoveFile(files);
      },
      handleRemoveFile(files){
        postAction(this.url.removeFile, {filePaths:files});
      },
    },
  }
</script>