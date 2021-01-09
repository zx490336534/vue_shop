<template>
  <div>
    <!-- 面包屑导航区 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!--卡片视图-->
    <el-card>
      <!--添加角色按钮区域-->
      <el-row>
        <el-col>
          <el-button type="primary" @click="addroleVisible=true">添加角色</el-button>
        </el-col>
      </el-row>
      <!--角色列表区域-->
      <el-table :data="rolelist" border stripe>
        <!--展开列-->
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row :class="['bdbottom',i1===0?'bdtop':'','vcenter']" v-for="(item1,i1) in scope.row.children"
                    :key="item1.id">
              <!-- 渲染一级权限-->
              <el-col :span="5">
                <el-tag closable @close="removeRightById(scope.row,item1.id)">
                  {{item1.authName}}
                </el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 渲染二级和三级权限-->
              <el-col :span="19">
                <!--通过for循环嵌套渲染二级权限-->
                <el-row :class="[i2===0?'':'bdtop','vcenter']" v-for="(item2,i2) in item1.children" :key="item2.id">
                  <el-col :span="6">
                    <el-tag type="success" closable @close="removeRightById(scope.row,item2.id)">
                      {{item2.authName}}
                    </el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="18">
                    <el-tag v-for="(item3,i3) in item2.children" :key="item3.id" type="warning" closable
                            @close="removeRightById(scope.row,item3.id)">
                      {{item3.authName}}
                    </el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <!--索引列-->
        <el-table-column type="index"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作" width="300px">
          <template slot-scope="scope">
            <el-button type="primary" icon="el-icon-edit" size="mini">编辑</el-button>
            <el-button type="danger" icon="el-icon-delete" size="mini">删除</el-button>
            <el-button type="warning" icon="el-icon-setting" size="mini" @click="showSetRightDialog(scope.row)">
              分配权限
            </el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>

    <!--分配权限的对话框-->
    <el-dialog title="分配权限" :visible.sync="setRightDialogVisible" width="50%" @close="setRightDialogClosed">
      <!--树形控件-->
      <el-tree :data="rightsList"
               :props="treeProps"
               show-checkbox
               node-key="id"
               default-expand-all
               :default-checked-keys="defKeys"
               ref="treeRef"
      ></el-tree>

      <span slot="footer" class="dialog-footer">
        <el-button @click="setRightDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="allotRights">确 定</el-button>
      </span>
    </el-dialog>
    <!--添加角色的对话框-->
    <el-dialog title="添加角色" :visible.sync="addroleVisible" width="50%" @close="addroleDialogClosed">
      <el-form :model="roleInfo" :rules="addroleRules" ref="addroleRef" label-width="100px">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="roleInfo.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色表述">
          <el-input v-model="roleInfo.roleDesc"></el-input>
        </el-form-item>

      </el-form>
      <span slot="footer" class="dialog-footer">
          <el-button @click="addroleVisible = false">取 消</el-button>
          <el-button type="primary" @click="addrole">确 定</el-button>
        </span>
    </el-dialog>
  </div>
</template>

<script>
  export default {
    name: 'Roles',
    data() {
      return {
        // 所有角色列表数据
        rolelist: [],
        // 控制分配权限对话框的显示与隐藏
        setRightDialogVisible: false,
        // 所有权限的数据
        rightslist: [],
        // 树形控件的属性绑定对象
        treeProps: {
          label: 'authName',
          children: 'children'
        },
        // 默认选中的节点ID
        defKeys: [],
        //当前即将分配权限的角色id
        roleId: '',
        //添加角色表单内容
        roleInfo: {
          roleName: '',
          roleDesc: ''
        },
        // 添加角色对话框的显示与隐藏
        addroleVisible: false,
        addroleRules: {
          roleName: [
            { required: true, message: '请输入角色名称', trigger: 'blur' }
          ]

        }
      }
    },
    created() {
      this.getRolesList()
    },
    methods: {
      // 获取所有角色列表
      async getRolesList() {
        const { data: res } = await this.$http.get('roles')
        if (res.meta.status !== 200) {
          return this.$message.error('获取角色列表失败')
        }
        this.rolelist = res.data
      },
      //根据ID删除对应的权限
      async removeRightById(role, rightId) {
        const confirmResult = await this.$confirm('此操作将永久删除该权限, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err => err)
        if (confirmResult !== 'confirm') {
          return this.$message.info('取消了删除！')
        }
        const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
        if (res.meta.status !== 200) {
          return this.$message.error('删除权限失败')
        }
        role.children = res.data
      },
      //展示分配权限的对话框
      async showSetRightDialog(role) {
        this.roleId = role.id
        //获取所有权限数据
        const { data: res } = await this.$http.get('rights/tree')
        if (res.meta.status !== 200) {
          return this.$message.error('获取权限数据失败！')
        }
        // 把获取到的权限数据保存到data中
        this.rightsList = res.data
        this.getLeafKeys(role, this.defKeys)
        this.setRightDialogVisible = true
      },
      // 通过递归的形式，获取角色下所有三级权限id，并保存到defKeys
      getLeafKeys(node, arr) {
        if (!node.children) {
          return arr.push(node.id)
        }
        node.children.forEach(item => {
          this.getLeafKeys(item, arr)
        })
      },
      // 监听分配权限对话框的关闭事件
      setRightDialogClosed() {
        this.defKeys = []
      },
      //点击为角色分配权限
      async allotRights() {
        const keys = [
          ...this.$refs.treeRef.getCheckedKeys(),
          ...this.$refs.treeRef.getHalfCheckedKeys()
        ]
        const idStr = keys.join(',')
        const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, { rids: idStr })
        if (res.meta.status !== 200) {
          return this.$message.error('分配权限失败！')
        }
        this.$message.success('分配权限成功！')
        this.getRolesList()
        this.setRightDialogVisible = false
      },
      //添加角色
      async addrole() {
        this.addroleVisible = true
        this.$refs.addroleRef.validate(async valid => {
          if (!valid) return
          const { data: res } = await this.$http.post('roles', {
            roleName: this.roleInfo.roleName,
            roleDesc: this.roleInfo.roleDesc
          })
          console.log(res.meta.status)
          if (res.meta.status !== 201) return this.$message.error('添加角色失败！')
          this.getRolesList()
          this.addroleVisible = false
          this.$message.success('添加角色成功！')
        })

      },
      //监听添加角色对话框关闭
      addroleDialogClosed() {
        this.roleInfo = {
          roleName: '',
          roleDesc: ''
        }
      }

    }
  }
</script>

<style lang="less" scoped>
  .el-tag {
    margin: 7px;
  }

  .bdtop {
    border-top: 1px solid #eee;
  }

  .bdbottom {
    border-bottom: 1px solid #eee;
  }

  .vcenter {
    display: flex;
    align-items: center;
  }
</style>
