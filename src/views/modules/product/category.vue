<!--  -->
<template>
  <div>
    <el-switch
      v-model="draggable"
      active-text="开启拖拽"
      inactive-text="关闭拖拽">
    </el-switch>
    <el-button v-if="draggable" @click="batchSave" >批量添加</el-button>

    <el-tree :data="menus"
             :props="defaultProps"
             :expand-on-click-node="false"
             show-checkbox
             node-key="catId"
             :default-expanded-keys="expandedKey"
             :draggable="draggable"
             :allow-drop="allowDrop"
             @node-drop="handleDrop"
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
             @click="edit(data)"
           >
             edit
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
      :title="title"
      :visible.sync="dialogVisible"
      width="30%"
      :close-on-click-modal="false"
      >
      <el-form :model="category">
        <el-form-item label="分类名称">
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
    <el-button type="primary" @click="submitData()">确 定</el-button>
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
      pCid: [],
      draggable: true,
      updateNodes: [],
      mexLevel: 0,
      title: '',
      dialogType: '',  // 输入框的类型
      category: {name: '', parentCid: 0, catLevel: 0, showStatus: 1, sort: 0, catId: null, productUnit: '', icon: ''},
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
    submitData () {
      if (this.dialogType === 'add') {
        this.addCategory()
      }
      if (this.dialogType === 'edit') {
        this.editCategory()
      }
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
        //  将之前删除的位置展开
        this.expandedKey = [this.category.parentCid]
        // 清空数据
        this.category = []
        // 重新刷星页面
        this.getmenus()
        if (data && data.code === 0) {
          this.$message({
            type: 'success',
            message: '保存成功!'
          })
        }
      })
    },
    edit (data) {
      this.title = '修改操作'
      this.dialogType = 'edit'
      this.dialogVisible = true
      console.log('要修改的数据是', data)
      //  发送请求 请求新的数据
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: 'get'
      }).then(({data}) => {
        this.category.name = data.data.name
        this.category.catId = data.data.catId
        this.category.icon = data.data.icon
        this.category.productUnit = data.data.productUnit
        this.category.parentCid = data.data.parentCid
      })
    },
    append (data) {
      this.title = '添加操作'
      this.dialogType = 'add'
      this.dialogVisible = true
      this.category.parentCid = data.parentCid
      this.category.catLevel = data.catLevel * 1 + 1
      this.category.catId = null
      this.category.icon = null
      this.category.productUnit = null
      this.category.sort = 0
      this.category.showStatus = 1
      this.category.name = ''
      console.log(data)
    },
    editCategory () {
      var {catId, name, icon, productUnit} = this.category
      var data = {catId: catId, name: name, icon: icon, productUnit: productUnit}
      this.$http({
        url: this.$http.adornUrl('/product/category/update'),
        method: 'post',
        data: this.$http.adornData(data, false)
      }).then(({data}) => {
        this.dialogVisible = false
        //  将之前删除的位置展开
        this.expandedKey = [this.category.parentCid]
        // 清空数据
        this.category = []
        // 重新刷星页面
        this.getmenus()
        this.$message({
          type: 'success',
          message: '保存成功!'
        })
      })
    },
    allowDrop (draggingNode, dropNode, type) {
      console.log('移动的数据是', draggingNode)
     // 被拖动的当前节点 和 托去父节点总层数 不能大于三  求出最大的深度
      this.countNodeLevel(draggingNode)
      let deep = Math.abs(this.mexLevel - draggingNode.level) + 1
      console.log('深度', deep)
      if (type === 'inner') {
        return (deep + dropNode.level) <= 3
      } else {
        return (deep + dropNode.parent.level) <= 3
      }
    },
    countNodeLevel (node) {
      if (node.childNodes != null && node.childNodes.length > 0) {
        for (let i = 0; i < node.childNodes.length; i++) {
          if (node.childNodes[i].level > this.mexLevel) {
            this.mexLevel = node.childNodes[i].level
          }
          this.countNodeLevel(node.childNodes[i])
        }
      }
    },
    handleDrop (draggingNode, dropNode, dropType, ev) {
      console.log('tree drop: ', dropNode.label, dropType)
    //  得到父亲结点的 Id
      // eslint-disable-next-line no-unused-vars
      let siblings = null
      // eslint-disable-next-line no-unused-vars
      let pCid = null
      if (dropType === 'before' || dropType === 'after') {
        pCid = dropNode.parent.data.catId === undefined ? 0 : dropNode.parent.data.catId
        siblings = dropNode.parent.childNodes
      } else {
        pCid = dropNode.data.catId
        siblings = dropNode.childNodes
      }
      this.pCid.push(pCid)
    //  最新顺序 放到那个里面就 重新排序那个
      for (let i = 0; i < siblings.length; i++) {
        if (siblings[i].data.catId === draggingNode.data.catId) {
          let catLevel = draggingNode.level
          // 层级变化
          catLevel = siblings[i].level
          //  修改子节点的层级  先修改层级
          this.updatedNodeLevel(siblings[i])
          // 如果便利的是自己 就变化父 id  层级没有改变 就使用默认的层级 在修改
          this.updateNodes.push({catId: siblings[i].data.catId, sort: i, parentCid: pCid, catLevel: catLevel})
        } else {
          // 兄弟 们只需要修改顺序
          this.updateNodes.push({catId: siblings[i].data.catId, sort: i})
        }
      }
      console.log('需要修改的位置是', this.updateNodes)
    },
    // 修改子节点的层级
    updatedNodeLevel (node) {
      if (node.childNodes.length > 0) {
        for (let i = 0; i < node.childNodes.length; i++) {
          var cNode = node.childNodes[i].data
          this.updateNodes.push({catId: cNode.catId, catLevel: node.childNodes[i].level})
          this.updatedNodeLevel(node.childNodes[i])
        }
      }
    },
    batchSave () {
      //  最新层级 执行更新操作 点击保存按钮 执行保存操作
      this.$http({
        url: this.$http.adornUrl('/product/category/update/sort'),
        method: 'post',
        data: this.$http.adornData(this.updateNodes, false)
      }).then(({data}) => {
        if (data && data.code === 0) {
          this.$message({
            type: 'success',
            message: '菜单修改成功!'
          })
          //  刷新菜单
          this.getmenus()
          //  展示默认的菜单
          this.expandedKey = this.pCid
          //  清空默认值
          this.updateNodes = []
          this.mexLevel = 0
          // this.pCid = 0
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
