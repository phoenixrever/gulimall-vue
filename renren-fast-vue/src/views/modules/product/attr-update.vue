<template>
  <el-dialog
    :close-on-click-modal="false"
    :visible.sync="visible"
    append-to-body
  >
    <el-row :gutter="20">
      <el-col>
        <el-tabs tab-position="left" style="width:98%">
          <el-tab-pane
            :label="group.attrGroupName"
            v-for="(group, gidx) in dataResp.attrGroups"
            :key="group.attrGroupId"
          >
            <!-- 遍历属性,每个tab-pane对应一个表单，每个属性是一个表单项  spu.baseAttrs[0] = [{attrId:xx,val:}]-->
            <el-form ref="form" :model="dataResp">
              <el-form-item
                :label="attr.attrName"
                v-for="(attr, aidx) in group.attrs"
                :key="attr.attrId"
              >
                <el-input
                  v-model="dataResp.baseAttrs[gidx][aidx].attrId"
                  type="hidden"
                  v-show="false"
                ></el-input>
                <el-select
                  v-model="dataResp.baseAttrs[gidx][aidx].attrValues"
                  :multiple="attr.valueType == 1"
                  filterable
                  allow-create
                  default-first-option
                  placeholder="请选择或输入值"
                >
                  <el-option
                    v-for="(val, vidx) in attr.valueSelect.split(';')"
                    :key="vidx"
                    :label="val"
                    :value="val"
                  ></el-option>
                </el-select>
                <el-checkbox
                  v-model="dataResp.baseAttrs[gidx][aidx].showDesc"
                  :true-label="1"
                  :false-label="0"
                  >快速展示</el-checkbox
                >
              </el-form-item>
            </el-form>
          </el-tab-pane>
        </el-tabs>
      </el-col>
    </el-row>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="submitSpuAttrs">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
export default {
  data() {
    return {
      visible: false,
      spuId: "",
      catalogId: "",
      dataResp: {
        //后台返回的所有数据
        attrGroups: [],
        baseAttrs: []
      },
      spuAttrsMap: {}
    };
  },
  computed: {},
  methods: {
    init(id, catalogId) {
      this.visible = true;
      this.clearData();
      this.spuId = id;
      this.catalogId = catalogId;
      if (this.spuId && this.catalogId) {
        this.showBaseAttrs();
        this.getSpuBaseAttrs();
      }
    },
    clearData() {
      this.dataResp.attrGroups = [];
      this.dataResp.baseAttrs = [];
      this.spuAttrsMap = {};
    },
    getSpuBaseAttrs() {
      this.visible = true;
      this.$http({
        url: this.$http.adornUrl(`/product/attr/base/listforspu/${this.spuId}`),
        method: "get"
      }).then(({ data }) => {
        data.data.forEach(item => {
          this.spuAttrsMap["" + item.attrId] = item;
        });
        console.log("~~~~", this.spuAttrsMap);
      });
    },
    getQueryParams() {
      this.spuId = this.$route.query.spuId;
      this.catalogId = this.$route.query.catalogId;
      console.log("----", this.spuId, this.catalogId);
    },
    showBaseAttrs() {
      let _this = this;
      this.$http({
        url: this.$http.adornUrl(
          `/product/attrgroup/${this.catalogId}/withattr`
        ),
        method: "get",
        params: this.$http.adornParams({})
      }).then(({ data }) => {
        console.log("------", data);
        //先对表单的baseAttrs进行初始化
        data.data.forEach(item => {
          let attrArray = [];
          item.attrs.forEach(attr => {
            let v = "";
            if (_this.spuAttrsMap["" + attr.attrId]) {
              v = _this.spuAttrsMap["" + attr.attrId].attrValue.split(";");
              //单选为字符串
              if (attr.valueType == 0) {
                v = v[0] + "";
              }
            }
            console.log(v);
            attrArray.push({
              attrId: attr.attrId,
              attrName: attr.attrName,
              attrValues: v,
              showDesc: _this.spuAttrsMap["" + attr.attrId]
                ? _this.spuAttrsMap["" + attr.attrId].quickShow
                : attr.showDesc
            });
          });
          this.dataResp.baseAttrs.push(attrArray);
        });
        console.log("attrArray------", this.dataResp);
        console.log("attrGroups------", data.data);
        this.dataResp.attrGroups = data.data;
      });
    },
    submitSpuAttrs() {
      console.log("·····", this.dataResp.baseAttrs);
      //spu_id  attr_id  attr_name             attr_value             attr_sort  quick_show
      let submitData = [];
      this.dataResp.baseAttrs.forEach(item => {
        item.forEach(attr => {
          let val = "";
          if (attr.attrValues instanceof Array) {
            val = attr.attrValues.join(";");
          } else {
            val = attr.attrValues;
          }

          if (val != "") {
            submitData.push({
              attrId: attr.attrId,
              attrName: attr.attrName,
              attrValue: val,
              quickShow: attr.showDesc
            });
          }
        });
      });

      this.$confirm("修改商品规格信息, 是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl(`/product/attr/update/${this.spuId}`),
            method: "post",
            data: this.$http.adornData(submitData, false)
          }).then(({ data }) => {
            this.$message({
              type: "success",
              message: "属性修改成功!"
            });
            this.visible = false;
          });
        })
        .catch(e => {
          this.$message({
            type: "info",
            message: "已取消修改" + e
          });
          this.visible = false;
        });
    }
  },
  created() {}
  //   activated() {
  //     this.clearData();
  //     this.getQueryParams();
  //     if (this.spuId && this.catalogId) {
  //       this.showBaseAttrs();
  //       this.getSpuBaseAttrs();
  //     }
  //   }
};
</script>
