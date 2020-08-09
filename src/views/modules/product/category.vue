<template>
  <el-tree
    :data="menus"
    show-checkbox
    node-key="catId"
    :props="defaultProps"
    @node-click="handleNodeClick"
    :expand-on-click-node="false"
    :default-expanded-keys="expandedKey"
  >
    <span class="custom-tree-node" slot-scope="{ node, data }">
      <span>{{ node.label }}</span>
      <span>
        <el-button type="text" size="mini" @click="() => append(data)">Append</el-button>
        <el-button type="text" size="mini" @click="() => remove(node, data)">Delete</el-button>
        <el-button type="text" size="mini" @click="() => edit(node, data)">Edit</el-button>
        <el-dialog :title="title" :visible.sync="dialogVisible" width="50%">
          <el-form :model="category">
            <el-form-item label="商品名称">
              <el-input v-model="category.name" autocomplete="off"></el-input>
            </el-form-item>
            <el-form-item label="图标">
              <el-input v-model="category.icon" autocomplete="off"></el-input>
            </el-form-item>
            <el-form-item label="计量单位">
              <el-input v-model="category.productUnit" autocomplete="off"></el-input>
            </el-form-item>
          </el-form>

          <span slot="footer" class="dialog-footer">
            <el-button @click="dialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="submitData">确 定</el-button>
          </span>
        </el-dialog>
      </span>
    </span>
  </el-tree>
</template>

<script>
export default {
  data() {
    return {
      menus: [],
      category: {
        name: "",
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        catId: null,
        icon: "",
        productUnit: ""
      },
      dialogVisible: false,
      dialogType: "",
      title: "",
      expandedKey: [],
      defaultProps: {
        children: "children",
        label: "name"
      }
    };
  },
  activated() {},
  created() {
    this.getMenus();
  },
  methods: {
    handleNodeClick(data) {
      console.log(data);
    },
    append(data) {
      this.dialogVisible = true;
      this.dialogType = "append";
      this.title = "append";
      this.category.parentCid = data.catId;
      this.category.catLevel = data.catLevel * 1 + 1;
      console.log("append data", data);
    },
    edit(data) {
      this.dialogVisible = true;
      this.dialogType = "edit";
      this.title = "edit";
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.data.catId}`),
        method: "get"
      }).then(({ data }) => {
        this.category.name = data.category.name;
        this.category.catId = data.category.catId;
        this.category.icon = data.category.icon;
        this.category.productUnit = data.category.productUnit;
        //需要获取这两个参数，否则无法修改后正常展开
        this.category.parentCid = data.category.parentCid;
        this.category.catLevel = data.category.catLevel;
        console.log("get data ok ", data);
      });
    },
    submitData() {
      if (this.dialogType == "append") {
        this.addCategory();
      } else if (this.dialogType == "edit") {
        this.editCategory();
      }
    },
    addCategory() {
      this.$http({
        url: this.$http.adornUrl("/product/category/save"),
        method: "post",
        data: this.$http.adornData(this.category, false)
      }).then(({ data }) => {
        if (data && data.code === 0) {
          this.$message({
            message: "操作成功",
            type: "success",
            duration: 1500
          });
          this.dialogVisible = false;
          this.getMenus();
          this.expandedKey = [this.category.parentCid];
        } else {
          this.$message.error(data.msg);
        }
      });

      console.log("add category", this.category);
    },

    editCategory() {
      var { name, catId, icon, productUnit } = this.category;
      this.$http({
        url: this.$http.adornUrl("/product/category/update"),
        method: "post",
        data: this.$http.adornData({ name, catId, icon, productUnit }, false)
      }).then(({ data }) => {
        if (data && data.code === 0) {
          this.$message({
            message: "修改操作成功",
            type: "success",
            duration: 1500
          });
          this.dialogVisible = false;
          this.getMenus();
          this.expandedKey = [this.category.parentCid];
        } else {
          this.$message.error(data.msg);
        }
      });

      console.log("update category", this.category);
    },

    remove(node, data) {
      var ids = [data.catId];
      this.$confirm(`此操作将永久删除${data.name}, 是否继续?`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(ids, false)
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.$message({
                message: "操作成功",
                type: "success",
                duration: 1500,
                onClose: () => {
                  this.getDataList();
                }
              });
              this.getMenus();
              this.expandedKey = [node.parent.data.catId];
            } else {
              this.$message.error(data.msg);
            }
          });
          this.$message({
            type: "success",
            message: "删除成功!"
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除"
          });
        });

      console.log("remove data node", node, data);
    },
    getMenus() {
      this.$http({
        url: this.$http.adornUrl("/product/category/list/tree"),
        method: "get"
      }).then(data => {
        console.log("get data successfully...", data.data);
        this.menus = data.data.entities;
      });
    }
  }
};
</script>
