<!--  -->
<template>
  <div>
    <el-tree
      :data="menus"
      :props="defaultProps"
      :expand-on-click-node="false"
      :default-expanded-keys="expandedkeys"
      show-checkbox
      false
      node-key="catId"
    >
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <!-- v-if="node.childNodes.length > 0" -->
          <el-button type="text" size="mini" @click="() => append(data)">
            新增
          </el-button>
          <el-button type="text" size="mini" @click="edit(data)">
            修改
          </el-button>
          <el-button
            v-if="node.childNodes.length == 0"
            type="text"
            size="mini"
            @click="() => remove(node, data)"
          >
            删除
          </el-button>
        </span>
      </span>
    </el-tree>

    <el-dialog :title="title" :visible.sync="dialogVisible" width="30%">
      <el-form :model="category">
        <el-form-item label="分类名称">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="submitData">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
//这里可以导入其他文件（比如：组件，工具js，第三方插件js，json文件，图片文件等等）
//例如：import 《组件名称》 from '《组件路径》';

export default {
  //import引入的组件需要注入到对象中才能使用
  components: {},
  data() {
    return {
      //定义一个记录父节点变量
      parentCid:null,
      //定义弹窗标题
      title: "",
      //定义一个标记，点击新增按钮则是add，点击修改则是edit
      dialogType: "",
      category: {
        catId: null,
        name: "",
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
      },
      dialogVisible: false,
      menus: [],
      expandedkeys: [], //默认展开的节点
      defaultProps: {
        children: "childrens",
        label: "name",
      },
    };
  },
  //监听属性 类似于data概念
  computed: {},
  //监控data中的数据变化
  watch: {},
  //方法集合
  methods: {
    //定义一个点击方法
    submitData() {
      if (this.dialogType == "add") {
        this.addCategory();
      }
      if (this.dialogType == "edit") {
        this.editCategory(this.data);
      }
    },

    //修改三级分类
    editCategory(data) {
      console.log(data)
      console.log("需要修改的数据", this.category);
      this.$http({
        url: this.$http.adornUrl("/product/category/update"),
        method: "post",
        data: this.$http.adornData(this.category, false),
      }).then(({ data }) => {
        this.$message({
          message: "分类修改成功",
          type: "success",
        });
        //刷新菜单
        this.dialogVisible = false;
        this.getDataList();
        //设置默认展开的菜单,当前删除节点父节点
        this.expandedkeys = [this.parentCid];
      });
    },
    //添加三级分类的方法
    addCategory() {
      console.log("三级分类", this.category);
      this.$http({
        url: this.$http.adornUrl("/product/category/save"),
        method: "post",
        data: this.$http.adornData(this.category, false),
      }).then(({ data }) => {
        this.$message({
          message: "分类新增成功",
          type: "success",
        });
        //刷新菜单
        this.dialogVisible = false;
        this.getDataList();
        //设置默认展开的菜单,当前删除节点父节点
        this.expandedkeys = [this.category.parentCid];
      });
    },
    append(data) {
      //每次进入清空
      Object.keys(this.category).forEach((key) => (this.category[key] = ""));

      console.log("data", data);
      this.dialogType = "add";
      this.title = "添加分类";

      this.dialogVisible = true;
      this.category.parentCid = data.catId;
      this.category.catLevel = data.catLevel * 1 + 1;
    },

    edit(data) {
      //每次进入清空
      Object.keys(this.category).forEach((key) => (this.category[key] = ""));

      console.log("修改的分类", data);
      this.dialogType = "edit";
      this.title = "修改分类";

      this.dialogVisible = true;
      this.category.name = data.name;
      this.category.catId = data.catId;
      this.parentCid = data.parentCid;
    },

    remove(node, data) {
      console.log("data", data);
      console.log("node", node);
      var ids = [data.catId];
      this.$confirm(`是否删除{${data.name}]菜单`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(ids, false),
          }).then(({ data }) => {
            this.$message({
              message: "菜单删除成功",
              type: "success",
            });
            //刷新菜单
            this.getDataList();
            //设置默认展开的菜单,当前删除节点父节点
            console.log(node.parent.data.catId);
            this.expandedkeys = [node.parent.data.catId];
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除",
          });
        });
    },

    handleNodeClick(data) {
      console.log(data);
    },
    // 获取数据列表
    getDataList() {
      this.dataListLoading = true;
      this.$http({
        url: this.$http.adornUrl("/product/category/list/tree"),
        method: "get",
      }).then(({ data }) => {
        console.log("成功获取菜单数据", data.listTree),
          (this.menus = data.listTree);
      });
    },
  },
  //生命周期 - 创建完成（可以访问当前this实例）
  created() {
    this.getDataList();
  },
  //生命周期 - 挂载完成（可以访问DOM元素）
  mounted() {},
  beforeCreate() {}, //生命周期 - 创建之前
  beforeMount() {}, //生命周期 - 挂载之前
  beforeUpdate() {}, //生命周期 - 更新之前
  updated() {}, //生命周期 - 更新之后
  beforeDestroy() {}, //生命周期 - 销毁之前
  destroyed() {}, //生命周期 - 销毁完成
  activated() {}, //如果页面有keep-alive缓存功能，这个函数会触发
};
</script> 
<style scoped>
</style> 