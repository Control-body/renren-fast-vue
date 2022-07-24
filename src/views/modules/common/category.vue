<!--  -->
<template>
      <el-tree :data="menus"
               :props="defaultProps"
               node-key="catId"
               ref="menuTree"
               @node-click="nodeClick"
      ></el-tree>
</template>

<script>
// import
export default {
  // 注册
  components: {},
  // 定义变量
  data () {
    return {
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
    nodeClick (data, node, component) {
      console.log('节点被点击', data, node, component)
      this.$emit('tree-node-click', data, node, component)
    }
  },
  // --------------------------- 生命周期 start -----------------------------------
  // 创建完成（可以访问当前this实例）
  created () {
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
