<template>
  <div>
    <!-- 面包屑导航区 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>参数列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!--卡片视图区域-->
    <el-card>
      <!--警告区-->
      <el-alert
        show-icon
        :closable="false"
        title="注意：只允许为第三级分类设置相关参数！"
        type="warning">
      </el-alert>
      <!--选择商品分类区域-->
      <el-row class="cat_opt">
        <el-col>
          <span>选择商品分类：</span>
          <!--选择商品分类的级联选择框-->
          <el-cascader
            v-model="selectedCateKeys"
            :options="catelist"
            :props="cateProps"
            @change="handleChange"></el-cascader>
        </el-col>
      </el-row>
      <!--tab页签区域-->
      <el-tabs v-model="activeName" @tab-click="handleTabClick">
        <!--添加动态参数的面板-->
        <el-tab-pane label="动态参数" name="first">
          <el-button type="primary" size="mini" :disabled="isBtnDiabled">添加参数</el-button>
        </el-tab-pane>
        <!--添加静态属性的面板-->
        <el-tab-pane label="静态属性" name="second">
          <el-button type="primary" size="mini" :disabled="isBtnDiabled">添加属性</el-button>
        </el-tab-pane>
      </el-tabs>
    </el-card>
  </div>
</template>

<script>
  export default {
    name: 'Params',
    data() {
      return {
        //商品分类列表
        catelist: [],
        //级联选择框对象
        cateProps: {
          value: 'cat_id',
          label: 'cat_name',
          children: 'children'
        },
        //级联选择框双向绑定到的数组
        selectedCateKeys: [],
        // 被激活的页签的名称
        activeName: 'first'
      }
    },
    computed: {
      isBtnDiabled() {
        if (this.selectedCateKeys.length !== 3) {
          return true
        }
        return false
      }
    },
    created() {
      this.getCateList()
    },
    methods: {
      //获取所有商品分类列表
      async getCateList() {
        const { data: res } = await this.$http.get('categories')
        if (res.meta.status !== 200) {
          return this.$message.error('获取商品分类列表失败！')
        }
        this.catelist = res.data

      },
      // 级联选择框选中值变化，会触发这个函数
      handleChange() {
        if (this.selectedCateKeys.length !== 3) {
          this.selectedCateKeys = []
        }
      },
      //Tab页签被点击
      handleTabClick() {

      }
    }

  }
</script>

<style lang="less" scoped>
  .cat_opt {
    margin: 15px 0;
  }
</style>
