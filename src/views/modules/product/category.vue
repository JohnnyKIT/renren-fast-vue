<template>
  <div class="category-list">
    <!--这是一棵树-->
    <el-tree
      :data="treeData"
      :props="treeProps"
      node-key="catId"
      :default-expanded-keys="expanedKeys"
      @node-click="handleNodeClick"
    >
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            type="text"
            size="mini"
            v-if="data.catLevel < 3"
            @click="() => appendBtn(data)"
          >
            添加
          </el-button>
          <el-button
            v-if="data.childrens.length == 0"
            type="text"
            size="mini"
            @click="() => removeBtn(node, data)"
          >
            删除
          </el-button>
        </span>
      </span>
    </el-tree>
  </div>
</template>

<script>
export default {
  data() {
    return {
      treeData: [],
      expanedKeys: [],
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
    handleNodeClick() {
      console.log("handleNodeClick");
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
    appendBtn() {
      console.log("appendBtn");
    },
    removeBtn(node, data) {
      let id = [data.catId];

      this.$confirm(`是否删除菜单【${data.name}】, 是否继续?`, "提示", {
        confirmButtonText: "删除",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl("product/category/delete"),
            method: "post",
            data: this.$http.adornData(id, false),
          }).then(({ data }) => {
            console.log(data);
            this.initTreeData();
            this.$message({
              type: "success",
              message: "删除成功!",
            });
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除",
          });
        });
    },
  },
};
</script>
