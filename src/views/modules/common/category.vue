<template>
  <el-tree :data="menus" node-key="catId" :props="defaultProps" @node-click="nodeclick"></el-tree>
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
    nodeclick(data, node, component) {
        console.log("category is clicked", data, node, component);
        this.$emit("tree-node-click", data, node, component)
    },
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
