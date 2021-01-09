<template>
  <div>
    <!-- 面包屑导航区 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary">添加分类</el-button>
        </el-col>
      </el-row>
      <!--表格-->
      <!--分页-->
    </el-card>
  </div>
</template>

<script>
  export default {
    name: 'Cate',
    data() {
      return {
        // 商品分类的数据列表
        catelist: [],
        // 查询条件
        querInfo: {
          type: 3,
          pagenum: 1,
          pagesize: 5
        },
        // 总数据条数
        total: 0
      }
    },
    created() {
      this.getCateList()
    },
    methods: {
      async getCateList() {
        const { data: res } = await this.$http.get('categories', { params: this.querInfo })
        if (res.meta.status !== 200) return this.$message.error('获取商品分类失败')
        this.catelist = res.data.result
        this.total = res.data.total
      }

    }
  }
</script>

<style lang="less" scoped>

</style>
