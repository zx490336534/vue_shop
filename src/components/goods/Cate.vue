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
          <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditCatDialog(scope.row)">编辑
          </el-button>
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

    <!--添加分类对话框-->
    <el-dialog
      title="添加分类"
      :visible.sync="addCatDialogVisible"
      width="50%"
      @close="addCateDialogClosed"
    >
      <!--添加分类的表单-->
      <el-form :model="addCateForm" :rules="addCateFormRules" ref="addCateFormRef" label-width="100px">
        <el-form-item label="分类名称:" prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类:">
          <!--options用来指定数据源-->
          <!--props用来指定配置对象-->
          <el-cascader
            v-model="selectedKeys"
            :options="parentCateList"
            :props="cascaderProps"
            @change="parentCateChanged"
            clearable
          ></el-cascader>
        </el-form-item>
      </el-form>

      <span slot="footer" class="dialog-footer">
        <el-button @click="addCatDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate">确 定</el-button>
      </span>
    </el-dialog>

    <!--编辑对话框-->
    <el-dialog
      title="编辑分类"
      :visible.sync="editCatDialogVisible"
      width="50%"
      @close="editCateDialogClosed"
    >
      <!--编辑分类的表单-->
      <el-form :model="editCateForm" :rules="addCateFormRules" ref="editCateFormRef" label-width="100px">
        <el-form-item label="分类名称:" prop="cat_name">
          <el-input v-model="editCateForm.cat_name"></el-input>
        </el-form-item>
      </el-form>

      <span slot="footer" class="dialog-footer">
        <el-button @click="editCatDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editCate">确 定</el-button>
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
        },
        // 父级分类列表
        parentCateList: [],
        // 指定级联选择器的配置对象
        cascaderProps: {
          value: 'cat_id',
          label: 'cat_name',
          children: 'children',
          checkStrictly: true
        },
        // 选中的父级分类的Id数组
        selectedKeys: [],
        // 控制编辑对话框显示与隐藏
        editCatDialogVisible: false,
        // 编辑分类序号
        catId: '',
        editCateForm: {
          cat_name: ''
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
        this.getParentCateList()
        this.addCatDialogVisible = true
      },
      //获取父级分类的数据列表
      async getParentCateList() {
        const { data: res } = await this.$http.get('categories', {
          params: {
            type: 2
          }
        })
        if (res.meta.status !== 200) {
          return this.$message.error('获取父级分类数据失败！')
        }
        this.parentCateList = res.data
      },
      // 选则项变化出发该函数
      async parentCateChanged() {
        if (this.selectedKeys.length > 0) {
          this.addCateForm.cat_pid = this.selectedKeys[this.selectedKeys.length - 1]
          this.addCateForm.cat_level = this.selectedKeys.length
          return
        } else {
          this.addCateForm.cat_pid = 0
          this.addCateForm.cat_level = 0
        }

      },
      addCateDialogClosed() {
        this.$refs.addCateFormRef.resetFields()
        this.selectedKeys = []
        this.addCateForm.cat_level = 0
        this.addCateForm.cat_pid = 0
      },
      // 点击按钮，添加新的分类
      addCate() {
        this.$refs.addCateFormRef.validate(async valid => {
          if (!valid) return
          const { data: res } = await this.$http.post('categories', this.addCateForm)
          if (res.meta.status !== 201) return this.$message.error('添加分类失败！')
          this.getCateList()
          this.addCatDialogVisible = false
          this.$message.success('添加分类成功！')
        })
      },
      showEditCatDialog(editrow) {
        this.editCatDialogVisible = true
        this.catId = editrow.cat_id
        this.editCateForm.cat_name = editrow.cat_name

      },
      editCate() {
        this.$refs.editCateFormRef.validate(async valid => {
          if (!valid) return
          const { data: res } = await this.$http.put(`categories/${this.catId}`, this.editCateForm)
          if (res.meta.status !== 200) return this.$message.error('编辑分类名称失败！')
          this.getCateList()
          this.editCatDialogVisible = false
          this.$message.success('编辑分类名称成功！')
        })
      },
      editCateDialogClosed() {
        this.$refs.editCateFormRef.resetFields()
        this.catId = ''
        this.editCateForm.cat_name = ''
      }

    }
  }
</script>

<style lang="less" scoped>
  .treeTable {
    margin-top: 15px;
  }

  .el-cascader {
    width: 100%;
  }
</style>
