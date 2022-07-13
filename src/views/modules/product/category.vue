<!--  -->
<template>
  <div>
    <el-tree :data="menus"
             :props="defaultProps"
             :expand-on-click-node="false"
             show-checkbox
             node-key="catId"
             :default-expanded-keys="expandedKey"
    >
       <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            type="text"
            size="mini"
            @click="() => append(data)"
            v-if="node.level <=2"
          >
            Append
          </el-button>
          <el-button
            type="text"
            size="mini"
            @click="() => remove(node, data)"
            v-if="node.childNodes.length ==0">
            Delete
          </el-button>
        </span>
      </span>

    </el-tree>

<!-- 编辑框-->
    <el-dialog
      title="提示"
      :visible.sync="dialogVisible"
      width="30%"
      >
      <el-form :model="category">
        <el-form-item label="分类名称">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
    <el-button @click="dialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="addCategory(da)">确 定</el-button>
  </span>
    </el-dialog>

  </div>
</template>

<script>
// import
export default {
  // 注册
  components: {},
  // 定义变量
  data () {
    return {
      category: {name: '', parentCid: 0, catLevel: 0, showStatus: 1, sort: 0},
      dialogVisible: false,
      menus: [],
      expandedKey: [],
      defaultProps: {
        children: 'children',
        label: 'name'
      }
    }
  },
  // 计算属性 当里面的数据发生变化时，会自动触发重新计算
  computed: {},
  // 监控data中的变量变化
  watch: {},
  // 方法集合
  methods: {
    getmenus () {
      this.$http({
        url: this.$http.adornUrl('/product/category/list/tree'),
        method: 'get'
      }).then(({data}) => {
        this.menus = data.data
        console.log('数据来咯！！！', data.data)
      })
    },
    append (data) {
      this.dialogVisible = true
      this.dialogVisible = true
      this.category.parentCid = data.catId
      this.category.catLevel = data.catLevel * 1 + 1
      console.log(data)
    },
    remove (node, data) {
      var ids = [data.catId]
      this.$confirm(`此操作将永久删除${data.name}, 是否继续?`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$http({
          url: this.$http.adornUrl('/product/category/delete'),
          method: 'post',
          data: this.$http.adornData(ids, false)
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.$message({
              type: 'success',
              message: '删除成功!'
            })
            // 重新刷星页面
            this.getmenus()
          //  将之前删除的位置展开
            this.expandedKey = [node.parent.data.catId]
          } else {
            this.$message.error(data.msg)
          }
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        })
      })
      console.log(node, data)
    },
    // 添加分类
    addCategory () {
      console.log(this.category)
      this.$http({
        url: this.$http.adornUrl('/product/category/save'),
        method: 'post',
        data: this.$http.adornData(this.category, false)
      }).then(({data}) => {
        // 关闭对话框
        this.dialogVisible = false
        this.expandedKey = [this.category.parentCid]
        // 清空数据
        this.category = []
        // 重新刷星页面
        this.getmenus()
        //  将之前删除的位置展开
        if (data && data.code === 0) {
          this.$message({
            type: 'success',
            message: '保存成功!'
          })
        }
      })
    }
  },
  // --------------------------- 生命周期 start -----------------------------------
  // 创建完成（可以访问当前this实例）
  created () {
    // 商品的信息
    this.getmenus()
  },
  // 挂载完成（可以访问DOM元素）
  mounted () {},
  // 创建之前
  beforeCreate () {},
  // 挂载之前
  beforeMount () {},
  // 更新之前
  beforeUpdate () {},
  // 更新之后
  updated () {},
  // 销毁之前
  beforeDestroy () {},
  // 销毁完成
  destroyed () {},
  // --------------------------- 生命周期 end -----------------------------------
  // 如果页面有keep-alive缓存功能，这个函数会触发
  activated () {}
}
</script>
<style lang='scss' scoped>
// @import url() 引入公共css类
</style>
