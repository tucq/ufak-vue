<template>
  <div>
    <a-tabs defaultActiveKey="1" size="small">
      <a-tab-pane key="1">
        <span slot="tab">一级规格</span>
        <a-row style="padding-bottom: 15px;">
          <a-col :span="22" style="padding-left: 10px">
            <a-input placeholder="请输入一级规格描述" v-model="specsTitleOne" style="width:200px;" />
          </a-col>
          <a-col :span="2" style="">
            <a-button type="primary" @click="handleAddSpecs(1)">
              <a-icon type="plus"/>
            </a-button>
          </a-col>
        </a-row>

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
                    v-if="item.isEdit"
                    placeholder="请输入规格名称"
                    :value="item.specsTitle"
                    @change="e => handleTitleChange(e.target.value,index,1)"/>
                  <template v-else>{{item.specsTitle}}</template>
                </template>
              </a-card-meta>
              <template class="ant-card-actions" slot="actions">
                <a-upload
                  :data="{'index': index}"
                  :showUploadList="false"
                  :headers="headers"
                  :action="uploadAction"
                  :beforeUpload="beforeUpload"
                  @change="handleImageChange"
                >
                  <a-icon type="upload"/>
                </a-upload>

                <a-icon :type="item.editIcon" @click="editIconClick(index,1)"/>
                <a-popconfirm title="确定删除吗?" @confirm="() => handelRemove(index,1)">
                  <a-icon type="close" />
                </a-popconfirm>
                <a-switch checkedChildren="开" unCheckedChildren="关" :checked="item.stats" @click="e => handelCheck(e,index,1)"/>
              </template>
            </a-card>
          </a-list-item>
        </a-list>
      </a-tab-pane>

      <a-tab-pane key="2">
        <span slot="tab">二级规格</span>
        <a-row style="padding-bottom: 15px;">
          <a-col :span="22" style="padding-left: 10px">
            <a-input placeholder="请输入二级规格描述" v-model="specsTitleTwo" style="width:200px;" />
          </a-col>
          <a-col :span="2" style="">
            <a-button type="primary" @click="handleAddSpecs(2)">
              <a-icon type="plus"/>
            </a-button>
          </a-col>
        </a-row>

        <a-list :grid="{ gutter: 10, column: 4 }" :dataSource="dataTwo">
          <a-list-item slot="renderItem" slot-scope="item, index">
            <a-card :bodyStyle="{'padding-top': '15px','padding-left':'10px','padding-right':'10px','padding-bottom':'15px'}">
              <a-card-meta>
                <template slot="description">
                  <a-input
                    v-if="item.isEdit"
                    placeholder="请输入规格名称"
                    :value="item.specsTitle"
                    @change="e => handleTitleChange(e.target.value,index,2)"/>
                  <template v-else>{{item.specsTitle}}</template>
                </template>
              </a-card-meta>
              <template class="ant-card-actions" slot="actions">
                <a-icon :type="item.editIcon" @click="editIconClick(index,2)"/>
                <a-popconfirm title="确定删除吗?" @confirm="() => handelRemove(index,2)">
                  <a-icon type="close" />
                </a-popconfirm>
                <a-switch checkedChildren="开" unCheckedChildren="关" :checked="item.stats" @click="e => handelCheck(e,index,2)"/>
              </template>
            </a-card>
          </a-list-item>
        </a-list>
      </a-tab-pane>
    </a-tabs>

  </div>
</template>

<script>
    import {httpAction, getAction, postAction} from '@/api/manage'
    import Vue from 'vue'
    import {ACCESS_TOKEN} from "@/store/mutation-types"

    export default {
//        props:{
//          dataOne: Array,
//          dataTwo: Array,
//        },
        data() {
            return {
              specsTitleOne: '',
              specsTitleTwo: '',
              dataOne: [],
              dataTwo: [],
                headers: {},
                url: {
                    fileUpload: window._CONFIG['domianURL'] + "/sys/common/upload",
                    removeFile: window._CONFIG['domianURL'] + "/sys/common/remove",
                },

            }
        },
        watch:{
           'dataOne':function (){
             this.$emit('dataOne', this.dataOne);
           },
          'dataTwo':function (){
            this.$emit('dataTwo', this.dataTwo);
          },
          'specsTitleOne':function () {
            this.$emit('specsTitleOne', this.specsTitleOne);
          },
          'specsTitleTwo':function () {
            this.$emit('specsTitleTwo', this.specsTitleTwo);
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
            handleAddSpecs(level) {
                if(level === 1){
                  this.dataOne.push({
                    isEdit: true,
                    stats: true,
                    editIcon: 'edit',
                    specsTitle:'',
                    specsImage:'/img/logo.07b1638a.svg',
                  });
                }else{
                  this.dataTwo.push({
                    isEdit: true,
                    stats: true,
                    editIcon: 'edit',
                    specsTitle:'',
                  });
                }
            },
            editIconClick(index,level) {
                if(level === 1){
                  this.dataOne[index].isEdit = !this.dataOne[index].isEdit;
                  this.dataOne[index].editIcon = this.dataOne[index].isEdit ? 'check' : 'edit';
                }else{
                  this.dataTwo[index].isEdit = !this.dataTwo[index].isEdit;
                  this.dataTwo[index].editIcon = this.dataTwo[index].isEdit ? 'check' : 'edit';
                }
            },
            handleTitleChange(value,index,level) {
                if(level === 1){
                  this.dataOne[index].specsTitle = value;
                }else{
                  this.dataTwo[index].specsTitle = value;
                }
            },
            handelCheck(check,index,level){
                if(level === 1){
                  this.dataOne[index].stats = check;
                }else{
                  this.dataTwo[index].stats = check;
                }
            },
            handelRemove(index,level){
              if(level === 1){
                this.dataOne.splice(index,1);
              }else{
                this.dataTwo.splice(index,1);
              }
            },
            handleImageChange(info) {
                if (info.file.status === 'uploading') {
                    this.loading = true;
                    return;
                }
                if (info.file.status === 'done') {
                  let idx = info.file.response.result
                  this.dataOne[idx].specsImage = window._CONFIG['domianURL'] + "/sys/common/view/" + info.file.response.message;
                }
            },
            beforeUpload(file) {
                const isJPG = file.type === 'image/jpeg';
                if (!isJPG) {
                    this.$message.error('请上传jpg格式图片！');
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
