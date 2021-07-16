<template>
  <div>
    <el-input :style="{'margin-bottom':'20px'}" placeholder="输入关键字进行过滤" v-model="filterText"></el-input>
    <el-tree
      :data="data"
      node-key="catId"
      :props="defaultProps"
      @node-click="handleNodeClick"
        :filter-node-method="filterNode"
      :highlight-current="true"
      ref="categoryTree"
    >
      ></el-tree
    >
  </div>
</template>

<script>
export default {
  data() {
    return {
       filterText: "",
      data: [],
      defaultProps: {
        children: "children",
        label: "name"
      }
    };
  },
   //计算属性 类似于data概念
  computed: {},
  //监控data中的数据变化
  watch: {
    filterText(val) {
      this.$refs.categoryTree.filter(val);
    }
  },
  created() {
    this.getMenu();
  },
  methods: {
     //树节点过滤
    filterNode(value, data) {
      if (!value) return true;
      return data.name.indexOf(value) !== -1;
    },
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
    handleNodeClick(data, Node, self) {
      this.$emit("treenodeclick", data, Node, self);
    }
  }
};
</script>

<style></style>
