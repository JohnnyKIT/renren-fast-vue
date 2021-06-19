<template>
  <div class="category-list">
    <!--这是一棵树-->
    <el-tree
      :data="treeData"
      props="{label:'',children:''}"
      @node-click="handleNodeClick"
    ></el-tree>
  </div>
</template>

<script>
export default {
  data() {
    return {
      treeData: [],
    };
  },
  created(){
      this.initTreeData()
  },
  methods: {
    handleNodeClick() {
      console.log("handleNodeClick");
    },
    initTreeData(){
        this.$http({
          url: this.$http.adornUrl('product/category/category/list/tree'),
          method: 'get'
        }).then(({data}) => {
          if (data && data.code === 0) {
              this.treeData = data.category
          } else {

          }
        })
    }
  },
};
</script>
