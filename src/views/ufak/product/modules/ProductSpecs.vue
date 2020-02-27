<template>
  <div class="product-specs">
    <a-form-item label="一级规格描述" :labelCol="labelCol" :wrapperCol="wrapperCol">
      <a-row>
        <a-col :span="12" style="padding-left: 10px">
          <a-input v-decorator="[ 'salesVolume', {}]" style="width:150px;"/>
        </a-col>
        <a-col :span="12">
          <a-button type="primary" @click="handleAddOne">
            <a-icon type="plus"/>
          </a-button>
        </a-col>
      </a-row>
    </a-form-item>

    <a-list :grid="{ gutter: 10, column: 4 }" :dataSource="dataOne">
      <a-list-item slot="renderItem" slot-scope="item, index">
        <a-card :bodyStyle="{'padding-top': '15px','padding-left':'10px','padding-right':'10px','padding-bottom':'15px'}">
          <img
            alt="图片"
            :src="item.specsImage"
            slot="cover"
          />
          <a-card-meta>
            <template slot="description">
              <a-input
                v-if="isEdit"
                :value="item.specsTitle"
                @change="e => handleTitleChange(e.target.value)"/>
              <template v-else>{{item.specsTitle}}</template>
            </template>
          </a-card-meta>
          <template class="ant-card-actions" slot="actions">
            <a-upload
              :data="{'jsonData': {
                'index':index,
                'item':item
              }}"
              :showUploadList="false"
              :headers="headers"
              :action="uploadAction"
              :beforeUpload="beforeUpload"
              @change="handleImageChange"
            >
              <a-icon type="upload"/>
            </a-upload>

<!--            <a-upload-->
<!--              :action="uploadAction"-->
<!--              :headers="headers"-->
<!--              @change="imageChange"-->
<!--            >-->
<!--              <a-icon type="upload"/>-->
<!--            </a-upload>-->


            <a-icon :type="editIcon" @click="editIconClick()"/>
            <a-icon type="close"/>
            <a-switch checkedChildren="开" unCheckedChildren="关" defaultChecked/>
            <!--          <a-icon :type="disabledIcon" @click="handelDisabled()" />-->
          </template>
        </a-card>
      </a-list-item>
    </a-list>
  </div>
</template>

<script>
    import {httpAction, getAction, postAction} from '@/api/manage'
    import Vue from 'vue'
    import {ACCESS_TOKEN} from "@/store/mutation-types"

    export default {
        data() {
            return {
                labelCol: {
                    xs: {span: 24},
                    sm: {span: 5},
                },
                wrapperCol: {
                    xs: {span: 24},
                    sm: {span: 16},
                },
                dataOne:[],
                editIcon: 'edit',
                isEdit: true,
                disabledIcon: 'check-circle',
                isDisabled: false,
                headers: {},
                previewVisible: false,
                previewImage: '',
                fileList: [],
                removeFileList: [],
                srcUrl: '',
                url: {
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
            handleAddOne() {
                this.dataOne.push({
                    specsTitle:'请输入规格名称',
                    specsImage:'~@/assets/logo.svg',
                });
            },
            editIconClick() {
                this.isEdit = !this.isEdit;
                this.editIcon = this.isEdit ? 'check' : 'edit';
            },
            handleTitleChange(value) {
                this.testTxt = value
            },
            // handelDisabled() {
            //     this.isDisabled = !this.isDisabled
            //     this.disabledIcon = this.isDisabled ? 'check-circle' : 'stop';
            // },
            handleImageChange(info) {
                if (info.file.status === 'uploading') {
                    this.loading = true;
                    return;
                }
                if (info.file.status === 'done') {
                    console.log(info);
                    // Get this url from response in real world.
                    // getBase64(info.file.originFileObj, imageUrl => {
                    //     this.dataOne[0].specsImage = imageUrl;
                    //     alert(this.dataOne[0].specsImage);
                    //     this.loading = false;
                    // });
                }
            },
            beforeUpload(file) {
                const isJPG = file.type === 'image/jpeg';
                if (!isJPG) {
                    this.$message.error('请上传图片！');
                }
                const isLt2M = file.size / 1024 / 1024 < 0.5;
                if (!isLt2M) {
                    this.$message.error('Image must smaller than 0.5MB!');
                }
                return isJPG && isLt2M;
            },
        },

    };
</script>
<style scoped>


</style>
