<template>
  <div class="category-list">
    <!-- 添加和编辑的模态框 -->
    <el-dialog
      :title="CategoryFormTitle"
      :visible.sync="updateVisible"
      width="30%"
      @close="closeCategoryDiaglog"
    >
      <el-form ref="category" :model="category" label-width="80px">
        <el-form-item label="分类名称">
          <el-input v-model="category.name"></el-input>
        </el-form-item>
        <el-form-item label="排序">
          <el-input v-model="category.sort"></el-input>
        </el-form-item>
        <el-form-item label="商品单位">
          <el-input v-model="category.productUnit"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="closeCategoryDiaglog">取 消</el-button>
        <el-button type="primary" @click="submitCategory()">确 定</el-button>
      </span>
    </el-dialog>

    <!--这是一棵树-->
    <el-tree
      :data="treeData"
      :props="treeProps"
      node-key="catId"
      :default-expanded-keys="expanedKeys"
      draggable
      :allow-drop="allowDrop"
      @node-drop="handleDrop"
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
          <el-button type="text" size="mini" @click="() => editBtn(data)">
            编辑
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
      updateVisible: false,
      category: {
        name: "",
        catId: null,
        parentCid: 0,
        catLevel: 0,
        sort: 0,
        showStatus: 1,
        productUnit: "",
      },
      CategoryFormTitle: "",
      maxDeep: 1, //用于计算节点深度的变量
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
    appendBtn(data) {
      console.log("appendBtn", data);
      this.category.parentCid = data.catId;
      this.category.catLevel = data.catLevel * 1 + 1;
      this.updateVisible = true;
      if (this.category.catId == null) {
        this.CategoryFormTitle = "添加分类";
      }
    },
    submitCategory() {
      if (this.category.catId == null) {
        this.addCategory();
      } else {
        this.updateCategory();
      }
    },
    addCategory() {
      console.log(this.category);
      this.$http({
        url: this.$http.adornUrl("product/category/save"),
        method: "post",
        data: this.$http.adornData(this.category, false),
      }).then(({ data }) => {
        this.$message({
          type: "success",
          message: "添加成功!",
        });
        this.expanedKeys = [this.category.parentCid];
        this.updateVisible = false;
        this.initTreeData();
      });
    },
    updateCategory() {
      this.$http({
        url: this.$http.adornUrl("product/category/update"),
        method: "post",
        data: this.$http.adornData(this.category, false),
      }).then(({ data }) => {
        this.$message({
          type: "success",
          message: "修改成功!",
        });
        this.expanedKeys = [this.category.catId];
        this.updateVisible = false;
        this.initTreeData();
      });
    },
    removeBtn(node, data) {
      console.log(node, data);
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
            this.expanedKeys = [node.parent.key];
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
    editBtn(data) {
      this.$http({
        url: this.$http.adornUrl(`product/category/info/${data.catId}`),
        method: "get",
        params: this.$http.adornParams({}),
      }).then(({ data }) => {
        console.log(data);
        this.category = data.category;
        this.updateVisible = true;
        if (this.category.catId != null) {
          this.CategoryFormTitle = "修改分类";
        }
      });
    },
    closeCategoryDiaglog() {
      this.category = {
        name: "",
        catId: null,
        parentCid: 0,
        catLevel: 0,
        sort: 0,
        showStatus: 1,
      };
      this.updateVisible = false;
    },

    allowDrop(draggingNode, dropNode, type) {
      //拖拽判断
      //console.log(draggingNode, dropNode, type);
      //计算深度
      let deep = this.calNodeDeep(draggingNode);
      this.maxDeep = 1;

      //console.log(draggingNode, deep);

      if (type == "inner") {
        return dropNode.level + deep < 4;
      } else {
        return dropNode.parent.level + deep < 4;
      }
    },
    calNodeDeep(Node) {
      //计算节点的深度
      if (Node.childNodes.length == 0) {
        return 1;
      }
      for (let i = 0; i < Node.childNodes.length; i++) {
        let item = Node.childNodes[i];
        if (item.level > this.maxDeep) {
          this.maxDeep = item.level;
        }
        this.calNodeDeep(item); //递归
      }
      return this.maxDeep - Node.level + 1;
    },
    handleDrop(draggingNode, dropNode, dropType, ev) {
      console.log(draggingNode, dropNode, dropType);
      // 需要修改的字段catLevel sort  parentCid
      //必须修改：自身节点以及新兄弟节点的排序
      //原兄弟不用修改
      //判断是否需要修改所有子节点的level
      let data = [];
      // if (draggingNode.level == dropNode.level) {
        //更改新级联拖拽
        let brother = dropType=='inner'?dropNode.childNodes:dropNode.parent.childNodes
        for (let i = 0; i < brother.length; i++) {
          let node = brother[i];
          //let catLevel = node.level;
          let catLevel = node.level;
          let sort = i;
          let catId = node.key;
          let parentCid =
            dropNode.parent.key == undefined ? 0 : dropNode.parent.key;
          data.push({ catLevel, sort, catId, parentCid });
        }
        if((dropType=='inner'||dropNode.level!=draggingNode.level)&draggingNode.childNodes.length>0){//利联发生变化 值需要修改所有子节点level
          let j = 0;
          if(dropType=='inner'){
            j = dropNode.level-draggingNode.level+1
          }else{
            j=dropNode.level-draggingNode.level
          }
          this.changeNodeLevel(draggingNode,j,data)
        }
      // } else {
      //   const j = dropNode.level - draggingNode.level;
      //   for (let i = 0; i < dropNode.childNodes.length; i++) {
      //     let node = dropNode.childNodes[i];
      //     //let catLevel = node.level;
      //     //let catLevel = dropType == "inner" ? dropNode.level - 1 : node.level;
      //     if(node.key==dropNode.key){
      //       this.changeNodeLevel(node,j,data)
      //     }
      //     let sort = i;
      //     let catId = node.key;
      //     let parentCid =
      //       dropNode.parent.key == undefined ? 0 : dropNode.parent.key;
          
      //     data.push({ catLevel, sort, catId, parentCid });
      //   }
        

      //}
      // if (draggingNode.level != dropNode.level) {
      //   //等级属性发生
      //   //let catLevel = dropType='inner'
      // }

      console.log(data);
    },
    changeNodeLevel(node,j,data){//递归改变Node节点和它下面的所有子节点的level +j,并放入data中
      for (let i = 0; i < node.childNodes.length; i++) {
        let child = node.childNodes[i];
        //console.log('child',child);
        let catLevel = child.level+j;
        let sort = i;
        let catId = child.key;
        let parentCid = node.key;
        this.changeNodeLevel(child,j,data);
        data.push({ catLevel, sort, catId, parentCid });
      }
    }

  },
};
</script>
