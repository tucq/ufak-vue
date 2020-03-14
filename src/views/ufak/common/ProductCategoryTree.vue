<template>
  <a-card :bordered="false">
    <div style="background: #fff;height: 100%; ">
       <!--<a-input-search @search="onSearch" style="width:100%;" placeholder="请输入分类名称"/>-->
      <template>
        <a-tree
          showLine
          :checkStrictly="true"
          :selectedKeys="selectedKeys"
          :autoExpandParent="autoExpandParent"
          :expandedKeys="iExpandedKeys"
          :treeData="goodsTypeTree"
          :dropdownStyle="{maxHeight:'200px',overflow:'auto'}"
          @expand="onExpand"
          @select="onSelect"
        />
      </template>
    </div>
  </a-card>
</template>

<script>
  import {searchByKeywords} from '@/api/api'
  import { getAction } from '@/api/manage'
  export default {
    name: "GoodsTypeTree",
    props: {
      queryChild:{
        type:Boolean,
        required:false,
        default:true
      },
      defaultSelectFirstNode:{
        type:Boolean,
        required:false,
        default:false
      },
      type:{
        type:String,
        required:false,
        default:"0"
      },
      loadUrl:{
        type:String,
        required:false,
        default:"/product/category/loadTypeTree"
      },
    },

    data() {
      return {
        description: '商品分类',
        loading: false,
        autoExpandParent: true,
        selectedKeys: [],
        goodsTypeTree: [],
        iExpandedKeys: [],
      }
    },
    created() {
      this.loadTree()
    },
    methods: {
      loadData() {
        this.refresh();
      },

      loadTree() {
        var that = this;
        let params = {
          type:that.type,
          queryChild:that.queryChild,
          async: false
        };
        that.treeData = [];
        that.goodsTypeTree = [];
        getAction(that.loadUrl,params).then((res) => {
          if (res.success) {
            for (let i = 0; i < res.result.length; i++) {
              let temp = res.result[i]
              that.treeData.push(temp)
              that.goodsTypeTree.push(temp)
              that.setThisExpandedKeys(temp)
            }
            if (that.defaultSelectFirstNode==true&&res.result.length>0){
              that.selectedKeys.push(res.result[0]["key"]);
              var e={
                node:{
                  dataRef:res.result[0]
                }
              }
              e.node.dataRef
              that.$emit("onSelect", that.selectedKeys, e);
            }
            this.loading = false
          }

        })
      },
      refresh() {
        this.loading = true
        this.loadTree()
      },
      onExpand(expandedKeys) {
        this.iExpandedKeys = expandedKeys
        this.autoExpandParent = false
      },
      onSearch(value) {
        let that = this
        if (value) {
          searchByKeywords({keyWord: value}).then((res) => {
            if (res.success) {
              that.departTree = []
              for (let i = 0; i < res.result.length; i++) {
                let temp = res.result[i]
                that.goodsTypeTree.push(temp)
              }
            } else {
              that.$message.warning(res.message)
            }
          })
        } else {
          that.loadTree()
        }

      },
      onSelect(selectedKeys, e) {
        if (this.selectedKeys[0] !== selectedKeys[0]) {
          this.selectedKeys = [selectedKeys[0]];
        }
        this.$emit("onSelect",selectedKeys, e);
      },
      setThisExpandedKeys(node) {
        if (node.children && node.children.length > 0) {
          this.iExpandedKeys.push(node.key)
          for (let a = 0; a < node.children.length; a++) {
            this.setThisExpandedKeys(node.children[a])
          }
        }
      },

    }
  }
</script>

<style scoped>

</style>