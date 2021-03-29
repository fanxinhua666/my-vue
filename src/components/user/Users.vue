<template>
  <div>
    <!-- 面包屑 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <!-- 栅格系统 -->
      <el-row :gutter="20">
        <el-col :span="8">
          <!-- 搜索区域 -->
          <el-input
            placeholder="请输入内容"
            v-model="queryInfo.query"
            clearable
            @clear="getUserList"
          >
            <el-button
              slot="append"
              icon="el-icon-search"
              @click="getUserList"
            ></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="showDialog">添加按钮</el-button>
        </el-col>
      </el-row>
      <!-- 表格 -->
      <el-table :data="userList" border stripe>
        <el-table-column type="index"></el-table-column>
        <el-table-column prop="username" label="姓名" width="120">
        </el-table-column>
        <el-table-column prop="email" label="邮箱"> </el-table-column>
        <el-table-column prop="mobile" label="手机" width="120">
        </el-table-column>
        <el-table-column prop="role_name" label="角色"> </el-table-column>
        <el-table-column label="状态">
          <template slot-scope="scope">
            <el-switch
              v-model="scope.row.mg_state"
              @change="userStateChanged(scope.row)"
            >
            </el-switch>
          </template>
        </el-table-column>
        <el-table-column label="操作" width="280">
          <template slot-scope="scope">
            <el-button
              type="primary"
              size="mini"
              icon="el-icon-edit"
              @click="showEditDialog(scope.row.id)"
            ></el-button>
            <el-button
              type="danger"
              size="mini"
              icon="el-icon-delete"
              @click="removeUserById(scope.row.id)"
            ></el-button>
            <el-tooltip effect="dark" content="分配角色" placement="top">
              <el-button
                type="warning"
                size="mini"
                icon="el-icon-setting"
              ></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[1, 2, 5, 10]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
    </el-card>
    <!-- 添加对话框 -->
    <el-dialog
      title="添加用户"
      :visible.sync="dialogVisibleAdd"
      width="50%"
      @close="addFormClose"
    >
      <el-form
        ref="addFormRef"
        :model="addForm"
        :rules="addFormRule"
        label-width="80px"
      >
        <el-form-item label="用户名" prop="username">
          <el-input v-model="addForm.username"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input v-model="addForm.password" type="password"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="addForm.email"></el-input>
        </el-form-item>
        <el-form-item label="手机号" prop="mobile">
          <el-input v-model="addForm.mobile"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisibleAdd = false">取 消</el-button>
        <el-button type="primary" @click="addUser">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改对话框 -->
    <el-dialog title="修改对话框" :visible.sync="dialogVisibleEdit" width="50%">
      <!-- 修改表单 -->
      <el-form
        :model="editForm"
        :rules="editFormRules"
        ref="editFormRef"
        label-width="70px"
        class="demo-ruleForm"
        @close="editClose"
      >
        <el-form-item label="用户名：">
          <el-input v-model="editForm.username" disabled></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="editForm.email"></el-input>
        </el-form-item>
        <el-form-item label="手机号" prop="mobile">
          <el-input v-model="editForm.mobile"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisibleEdit = false">取 消</el-button>
        <el-button type="primary" @click="editUserInfo">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    var checkEmail = (rule, value, cb) => {
      const regEmail = /^(\w-*\.*)+@(\w-?)+(\.\w{2,})+$/
      if (regEmail.test(value)) {
        return cb()
      }
      cb(new Error('请输入合法的邮箱'))
    }
    var checkMobile = (rule, value, cb) => {
      const regMobile = /^1[3|4|5|6|7|8|9]\d{9}$/
      if (regMobile.test(value)) {
        return cb()
      }
      cb(new Error('请输入合法的手机号'))
    }
    return {
      // 获取用户列表的参数对象
      queryInfo: {
        query: '',
        // 当前页
        pagenum: 1,
        // 一页几条数据
        pagesize: 2
      },
      // 用户列表
      userList: [],
      // 用户数量
      total: 0,
      // 添加用户对话框数据
      dialogVisibleAdd: false,
      // 修改对话框
      dialogVisibleEdit: false,
      // 添加用户的表单
      addForm: {
        username: '',
        password: '',
        mobile: '',
        email: ''
      },
      // 用户请求过来 的修改表单
      editForm: {},
      // 添加 用户表单的验证
      addFormRule: {
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          { min: 3, max: 10, message: '字符为3-10之间', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '请输入用户密码', trigger: 'blur' },
          { min: 6, max: 16, message: '密码字符6-16位', trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: '请输入手机号', trigger: 'blur' },
          { validator: checkMobile, trigger: 'blur' }
        ],
        email: [
          { required: true, message: '请输入用户邮箱', trigger: 'blur' },
          { validator: checkEmail, trigger: 'blur' }
        ]
      },
      // 修改用户的验证规则
      editFormRules: {
        email: [
          { required: true, message: '请输入用户邮箱', trigger: 'blur' },
          { validator: checkEmail, trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: '请输入手机号', trigger: 'blur' },
          { validator: checkMobile, trigger: 'blur' }
        ]
      },
      id: ''
    }
  },

  created() {
    this.getUserList()
  },

  methods: {
    // 获取用户列表
    async getUserList() {
      const { data: res } = await this.$http.get('users', {
        params: this.queryInfo
      })
      if (res.meta.status !== 200) {
        return this.$message.error('用户列表获取失败')
      }
      this.userList = res.data.users
      this.total = res.data.total
    },
    // 自定义索引列
    // indexMethod(index) {
    //   return index * 2
    // }
    // 监听pagesize事件
    handleSizeChange(newSize) {
      // console.log(newSize)
      this.queryInfo.pagesize = newSize
      this.getUserList()
    },
    // 监听 页码值 改变事件
    handleCurrentChange(newPage) {
      // console.log(newPage)
      this.queryInfo.pagenum = newPage
      this.getUserList()
    },
    // 监听switch开关的状态
    async userStateChanged(userinfo) {
      // console.log(userinfo)
      const { data: res } = await this.$http.put(
        `users/${userinfo.id}/state/${userinfo.mg_state}`
      )
      if (res.meta.status !== 200) {
        userinfo.mg_state = !userinfo.mg_state
        return this.$message.error('更新用户状态失败！')
      }
      this.$message.success('更新状态成功')
    },
    // 监听 添加对话框的关闭事件
    addFormClose() {
      this.$refs.addFormRef.resetFields()
    },
    // 验证成功后 添加用户
    addUser() {
      // console.log(this.$refs)
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) {
          return this.$message.error('验证用户时失败')
        }
        const { data: res } = await this.$http.post('users', {
          username: this.addForm.username,
          password: this.addForm.password,
          mobile: this.addForm.mobile,
          email: this.addForm.email
        })
        if (res.meta.status !== 201) return this.$message.error('添加用户失败')
        this.$message.success('用户添加成功')
        this.dialogVisibleAdd = false
        this.getUserList()
      })
    },
    // 显示添加对话框
    showDialog(id) {
      this.dialogVisibleAdd = true
      this.id = id
    },
    async showEditDialog(id) {
      this.dialogVisibleEdit = true
      const { data: res } = await this.$http.get('users/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('查询修改用户数据失败')
      }
      this.$message.success('查询用户数据修改成功')
      this.editForm = res.data
    },
    // 关闭对话框事件
    editClose() {
      this.$refs.editFormRef.resetFields()
    },
    // 修改用户对话框提交修改信息
    editUserInfo(id) {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) {
          return this.$message.error('验证失败')
        }
        const { data: res } = await this.$http.put(
          'users/' + this.editForm.id,
          this.editForm
        )
        if (res.meta.status !== 200) {
          return this.$message.error('修改用户数据错误')
        }
        this.$message.success('修改用户数据成功')
        this.getUserList()
        this.dialogVisibleEdit = false
      })
    },
    // 根据id删除用户数据
    async removeUserById(id) {
      const confirmresult = await this.$confirm(
        '此操作将永久删除该用户, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)
      // console.log(confirmresult)
      if (confirmresult !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      // console.log('确定了删除')
      const { data: res } = await this.$http.delete(
        'users/' + id,
        this.editForm
      )
      if (res.meta.status !== 200) {
        return this.$message.error('删除用户错误')
      }
      this.$message.success('删除用户成功')
      this.getUserList()
    }
  }
}
</script>
<style scoped lang="less"></style>
