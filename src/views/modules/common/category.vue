<template>
  <!-- 添加和编辑的模态框 -->

  <!--这是一棵树-->
  <el-tree
    :data="treeData"
    :props="treeProps"
    node-key="catId"
    @node-click="handleNodeClick"
    ref="tree"
  >
  </el-tree>
</template>

<script>
export default {
  data() {
    return {
      treeData: [],
      treeProps: {
        children: "childrens",
        label: "name",
      },
    };
  },
  created() {
    this.initTreeData();
  },
  methods: {
    handleNodeClick(data,node) {
      console.log("handleNodeClick=",data,node);
      this.$emit("node-click",data,node)
    },
    initTreeData() {
      this.$http({
        url: this.$http.adornUrl("product/category/list/tree"),
        method: "get",
      }).then(({ data }) => {
        if (data && data.code === 0) {
          console.log(data);
          this.treeData = data.category;
        } else {
        }
      });
    },
  },
};
</script>
