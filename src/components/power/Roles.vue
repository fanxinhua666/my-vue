<template>
  <div>
    <!-- 面包屑 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <!-- 添加角色按钮区域 -->
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAddRoles"> 添加角色</el-button>
        </el-col>
      </el-row>
      <!-- 表格区域 -->
      <el-table :data="rolesList" style="width: 100%" border stripe>
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row
              :class="['bdbottom', 'vcenter', index1 === 0 ? 'bdtop' : ' ']"
              v-for="(item1, index1) in scope.row.children"
              :key="item1.id"
            >
              <el-col :span="5">
                <el-tag
                  closable
                  @close="removeRightsById(scope.row, item1.id)"
                  >{{ item1.authName }}</el-tag
                >
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 渲染二级和三级权限 -->
              <el-col :span="17">
                <!-- 通过for循环 嵌套的渲染二级权限 -->
                <el-row
                  :class="[i2 === 0 ? '' : 'bdtop', 'vcenter']"
                  v-for="(item2, i2) in item1.children"
                  :key="item2.id"
                >
                  <el-col :span="6">
                    <el-tag
                      type="success"
                      closable
                      @close="removeRightsById(scope.row, item2.id)"
                      >{{ item2.authName }}</el-tag
                    >
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="18">
                    <el-tag
                      closable
                      @close="removeRightsById(scope.row, item3.id)"
                      type="warning"
                      v-for="item3 in item2.children"
                      :key="item3.id"
                    >
                      {{ item3.authName }}
                    </el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
            <!-- <pre>
              {{ scope.row }}
            </pre> -->
          </template>
        </el-table-column>
        <el-table-column type="index"> </el-table-column>
        <el-table-column prop="roleName" label="角色名称" width="180">
        </el-table-column>
        <el-table-column prop="roleDesc" label="角色描述"> </el-table-column>
        <el-table-column label="角色操作">
          <template slot-scope="scope">
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="mini"
              @click="showEditRoles(scope.row.id)"
              >编辑</el-button
            >
            <el-button
              type="danger"
              icon="el-icon-delete"
              size="mini"
              @click="removeRoles(scope.row.id)"
              >删除</el-button
            >
            <el-button
              type="warning"
              icon="el-icon-setting"
              size="mini"
              @click="showSetRightsDialog(scope.row)"
              >分配权限</el-button
            >
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!-- 添加角色对话框 -->
    <el-dialog
      title="添加角色"
      :visible.sync="addRolesDialogVisible"
      @close="closeRolesDialog"
      width="50%"
    >
      <!-- 添加角色表单 -->
      <el-form
        :model="rolesForm"
        :rules="rolesRules"
        ref="rolesFormRef"
        label-width="100px"
      >
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="rolesForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="rolesForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addRolesDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addRoles">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 编辑角色对话框 -->
    <el-dialog
      title="编辑角色"
      :visible.sync="editRolesDialogVisible"
      width="50%"
      @close="closeRolesDialog"
    >
      <!-- 编辑角色表单 -->
      <el-form
        :model="rolesForm"
        :rules="rolesRules"
        ref="rolesFormRef"
        label-width="100px"
      >
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="rolesForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="rolesForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editRolesDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editRoles">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 分配权限的对话框 -->
    <el-dialog
      title="tree权限数据列表"
      :visible.sync="setRightsDialogVisible"
      @close="closeSetRightsDialog"
      width="50%"
    >
      <!-- 树形控件 -->
      <el-tree
        :data="rightsList"
        :props="treeProps"
        show-checkbox
        node-key="id"
        default-expand-all
        :default-checked-keys="defKeys"
        ref="treeRef"
      ></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRightsDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="allotRights">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data() {
    return {
      // 所有的角色列表数据
      rolesList: [],
      // 添加角色对话框显示与隐藏
      addRolesDialogVisible: false,
      // 角色表单对象
      rolesForm: {
        roleName: '',
        roleDesc: ''
      },
      // 角色规则对象
      rolesRules: {
        roleName: [
          { required: true, message: '请输入角色', trigger: 'blur' },
          { min: 2, max: 6, message: '2-6位字符', trigger: 'blur' }
        ],
        roleDesc: [
          { required: true, message: '请输入描述', trigger: 'blur' },
          { min: 3, max: 20, message: '3-20位字符', trigger: 'blur' }
        ]
      },
      // 编辑角色显示隐藏
      editRolesDialogVisible: false,
      // 分配权限对话框显示与隐藏
      setRightsDialogVisible: false,
      // 所有权限的列表tree
      rightsList: [],
      // 树形控件对象
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      // 默认选择节点的id
      defKeys: [],
      // 分配角色id
      roleId: ''
    }
  },

  created() {
    // 自动加载角色数据列表
    this.getRolesList()
  },

  methods: {
    // 角色列表函数
    async getRolesList() {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色列表失败！')
      }
      // this.$message.success('获取角色列表成功！')
      this.rolesList = res.data
      // console.log(this.rolesList)
      // console.log('roles角色列表')
    },
    // 添加角色对话框显示
    showAddRoles() {
      this.addRolesDialogVisible = true
    },
    // 监听角色对话框的关闭事件
    closeRolesDialog() {
      // 点击关闭角色对话框，清空重置表单数据
      this.$refs.rolesFormRef.resetFields()
    },
    // 添加角色
    addRoles() {
      this.$refs.rolesFormRef.validate(async valid => {
        if (!valid) {
          return this.$message.error('添加角色验证失败')
        }
        const { data: res } = await this.$http.post('roles', this.rolesForm)
        if (res.meta.status !== 201) {
          return this.$message.error('添加角色失败')
        }
        this.$message.success('添加角色成功')
        this.addRolesDialogVisible = false
      })
    },
    // 显示编辑角色
    async showEditRoles(id) {
      this.id = id
      this.editRolesDialogVisible = true
      const { data: res } = await this.$http.get('roles/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('查询失败')
      }
      this.$message.success('查询成功')
      this.rolesForm = res.data
      // console.log(this.rolesForm)
    },
    // 修改角色
    editRoles() {
      this.$refs.rolesFormRef.validate(async valid => {
        if (!valid) {
          return this.$message.error('编辑角色验证失败')
        }
        const { data: res } = await this.$http.put(
          'roles/' + this.rolesForm.roleId,
          {
            roleDesc: this.rolesForm.roleDesc,
            roleName: this.rolesForm.roleName
          }
        )
        if (res.meta.status !== 200) {
          return this.$message.error('编辑角色失败')
        }
        this.$message.success('编辑角色成功')
        this.editRolesDialogVisible = false
        this.getRolesList()
      })
    },
    // 删除角色
    async removeRoles(id) {
      const confrimResult = await this.$confirm(
        '此操作将永久删除该角色, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)
      if (confrimResult !== 'confirm') {
        return this.$message.info('已取消删除该角色')
      }
      const { data: res } = await this.$http.delete('roles/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('删除角色失败')
      }
      this.$message.success('删除角色成功')
      this.getRolesList()
    },
    // 根据id删除权限 3级权限
    async removeRightsById(role, rightsId) {
      const confrimResult = await this.$confirm(
        '此操作将永久删除该权限, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)
      if (confrimResult !== 'confirm') {
        return this.$message.info('已取消删除权限')
      }
      const { data: res } = await this.$http.delete(
        `roles/${role.id}/rights/${rightsId}`
      )
      if (res.meta.status !== 200) {
        return this.$message.error('删除权限失败')
      }
      this.$message.success('删除权限成功')
      // 不建议重新获取全部的数据
      // this.getRolesList()
      role.children = res.data
    },
    // 显示分配权限的对话框
    async showSetRightsDialog(row) {
      this.roleId = row.id
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) {
        return this.$message.error('获取tree 权限数据列表失败')
      }
      this.rightsList = res.data
      console.log(this.rightsList)
      this.$message.success('获取权限列表成功')
      // 递归调用获取当前行的三级节点id
      this.getLeafKeys(row, this.defKeys)
      this.setRightsDialogVisible = true
    },
    // 通过递归的形式，获取角色下的所有三级权限的id并保持到defkeys[]数组中
    getLeafKeys(node, arr) {
      // 如果当前node节点不包含children属性，则是三级节点
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item => {
        this.getLeafKeys(item, arr)
      })
    },
    // 关闭分配权限对话框
    closeSetRightsDialog() {
      this.defKeys = []
    },
    // 分配角色权限
    async allotRights() {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      // console.log(keys)
      const idStr = keys.join(',')
      // console.log(idStr)
      const { data: res } = await this.$http.post(
        `roles/${this.roleId}/rights`,
        { rids: idStr }
      )
      if (res.meta.status !== 200) {
        return this.$message.error('分配权限失败')
      }
      this.$message.success('分配权限成功')
      this.getRolesList()
      this.setRightsDialogVisible = false
    }
  }
}
</script>

<style scoped lang="less">
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
