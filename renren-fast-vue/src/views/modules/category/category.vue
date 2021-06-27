<template>
  <el-tree
    :data="data"
    node-key="catId"
    :props="defaultProps"
    @node-click="handleNodeClick"
    ref="categoryTree"
  >
    ></el-tree
  >
</template>

<script>
export default {
  data() {
    return {
      data: [],
      defaultProps: {
        children: "children",
        label: "name"
      }
    };
  },
  created() {
    this.getMenu();
  },
  methods: {
    // 获取数据列表
    getMenu() {
      this.$http({
        url: this.$http.adornUrl("/product/category/list/tree"),
        method: "get",
        params: this.$http.adornParams()
      }).then(({ data }) => {
        // this.dataList = treeDataTranslate(data.list, 'catId', 'parentCid')
        this.data = data.list;
      });
    },
    handleNodeClick(data,Node,self) {
      console.log(Node,self)
      this.$emit("getCatId",data.catId)
    }
  }
};
</script>

<style></style>
