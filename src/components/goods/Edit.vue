<template>
  <div>
    <!-- 面包屑 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>编辑商品</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      edit
    </el-card>
  </div>
</template>

<script>
export default {
  data() {
    return {
      goodsObj: {}
    }
  },

  created() {
    this.getGoods()
  },

  methods: {
    // 查询商品
    async getGoods() {
      // console.log(location.hash)
      const str = location.hash.split('?')
      const str1 = str[1].split('=')
      const id = str1[1]
      console.log(str1[1])
      const { data: res } = await this.$http.get('goods/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('查询商品失败')
      }
      this.$message.success('查询商品成功')
      this.goodsObj = res.data
      console.log(this.goodsObj)
    }
  }
}
</script>

<style scoped lang="less"></style>
