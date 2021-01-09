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
          <el-button type="primary" @click="showAddCateDialog">添加分类</el-button>
        </el-col>
      </el-row>
      <!--表格-->
      <tree-table
        class="treeTable"
        :data="catelist"
        :columns="columns"
        :selection-type="false"
        :expand-type="false"
        show-index
        index-text="#"
        border
        :show-row-hover="false"
      >
        <!--是否有效-->
        <template slot="isok" slot-scope="scope">
          <i v-if="scope.row.cat_deleted===false" class="el-icon-success" style="color: lightgreen;"></i>
          <i v-else class="el-icon-error" style="color: red;"></i>
        </template>
        <!--排序-->
        <template slot="order" slot-scope="scope">
          <el-tag size="mini" v-if="scope.row.cat_level===0">一级</el-tag>
          <el-tag size="mini" type="success" v-else-if="scope.row.cat_level===1">二级</el-tag>
          <el-tag size="mini" type="warning" v-else>三级</el-tag>
        </template>
        <!--操作-->
        <template slot="opt" slot-scope="scope">
          <el-button type="primary" icon="el-icon-edit" size="mini">编辑</el-button>
          <el-button type="danger" icon="el-icon-delete" size="mini">删除</el-button>
        </template>

      </tree-table>
      <!--分页-->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="querInfo.pagenum"
        :page-sizes="[3, 5, 10, 15]"
        :page-size="querInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total">
      </el-pagination>
    </el-card>

    <el-dialog
      title="添加分类"
      :visible.sync="addCatDialogVisible"
      width="50%"
    >
      <!--添加分类的表单-->
      <el-form :model="addCateForm" :rules="addCateFormRules" ref="addCateFormRef" label-width="100px">
        <el-form-item label="分类名称:" prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类:">
        </el-form-item>
      </el-form>

      <span slot="footer" class="dialog-footer">
        <el-button @click="addCatDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCatDialogVisible = false">确 定</el-button>
      </span>
    </el-dialog>

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
        total: 0,
        columns: [
          {
            label: '分类名称',
            prop: 'cat_name'
          },
          {
            label: '是否有效',
            // 表示当前列定义为模版
            type: 'template',
            template: 'isok'
          },
          {
            label: '排序',
            // 表示当前列定义为模版
            type: 'template',
            template: 'order'
          },
          {
            label: '操作',
            // 表示当前列定义为模版
            type: 'template',
            template: 'opt'
          }
        ],
        //控制添加分类对话框的显示与隐藏
        addCatDialogVisible: false,
        //添加分类的表单数据对象
        addCateForm: {
          cat_name: '',
          cat_pid: 0,
          cat_level: 0
        },
        //添加分类表单的验证规则对象
        addCateFormRules: {
          cat_name: [
            { required: true, message: '请输入分类名称', trigger: 'blue' }
          ]
        }
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
      },
      //监听pagesize改变
      handleSizeChange(newSize) {
        this.querInfo.pagesize = newSize
        this.getCateList()
      },
      //监听pagenum改变
      handleCurrentChange(newPage) {
        this.querInfo.pagenum = newPage
        this.getCateList()
      },
      //点击按钮，展示添加分类的对话框
      showAddCateDialog() {
        this.addCatDialogVisible = true
      }

    }
  }
</script>

<style lang="less" scoped>
  .treeTable {
    margin-top: 15px;
  }
</style>
