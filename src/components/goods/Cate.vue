<template>
  <div>
    <!-- 面包屑 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAddCate">添加分类</el-button>
        </el-col>
      </el-row>
      <!-- 表格区域 -->
      <tree-table
        :data="cateList"
        :columns="columns"
        show-index
        :expand-type="false"
        :selection-type="false"
        index-text="#"
        border
        :show-row-hover="false"
      >
        <!-- 是否有效模板 -->
        <template slot="isok" slot-scope="scope">
          <i
            class="el-icon-success"
            v-if="scope.row.cat_deleted === false"
            style="color: lightgreen;"
          ></i>
          <i class="el-icon-error" v-else style="color: red;"></i>
        </template>
        <!-- 排序模板 -->
        <template slot="order" slot-scope="scope">
          <el-tag v-if="scope.row.cat_level === 0" size="mini">一级</el-tag>
          <el-tag
            v-else-if="scope.row.cat_level === 1"
            size="mini"
            type="success"
            >二级</el-tag
          >
          <el-tag v-else size="mini" type="warning">三级</el-tag>
        </template>
        <!-- 操作模板 -->
        <template slot="opt" slot-scope="scope">
          <el-button
            type="primary"
            size="mini"
            @click="showEditCate(scope.row.cat_id)"
            >编辑</el-button
          >
          <el-button
            type="danger"
            size="mini"
            @click="delCate(scope.row.cat_id)"
            >删除</el-button
          >
        </template>
      </tree-table>
      <!-- 分页区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[5, 10, 15, 20]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
    </el-card>
    <!-- 添加分类对话框 -->
    <el-dialog
      title="添加分类"
      :visible.sync="addCateDialogVisible"
      width="50%"
      @close="closeAddCateDialog"
    >
      <el-form
        :model="addCateForm"
        :rules="addCateRules"
        ref="addCateFormRef"
        label-width="100px"
      >
        <el-form-item label="分类名称" prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类">
          <el-cascader
            expand-trigger="hover"
            v-model="selectedKeys"
            :options="parentCateList"
            :props="cascaderProps"
            @change="parentCateChange"
            clearable
            change-on-select
          ></el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 编辑分类对话框 -->
    <el-dialog
      title="编辑分类"
      :visible.sync="editCateDialogVisible"
      width="50%"
      @close="closeAddCateDialog"
    >
      <el-form
        :model="addCateForm"
        :rules="addCateRules"
        ref="addCateFormRef"
        label-width="100px"
      >
        <el-form-item label="分类名称" prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <!-- <el-form-item label="父级分类">
          <el-cascader
            expand-trigger="hover"
            v-model="selectedKeys"
            :options="parentCateList"
            :props="cascaderProps"
            @change="parentCateChange"
            clearable
            change-on-select
          ></el-cascader>
        </el-form-item> -->
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editCate">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 获取商品分类数据
      cateList: [],
      // 查询条件
      queryInfo: {
        type: 3,
        // 页码值 第几页
        pagenum: 1,
        // 每页多少条
        pagesize: 5
      },
      // 商品数量总条数
      total: 0,
      columns: [
        {
          label: '分类名称',
          prop: 'cat_name'
        },
        {
          label: '是否有效',
          type: 'template',
          template: 'isok'
        },
        {
          label: '排序',
          type: 'template',
          template: 'order'
        },
        {
          label: '操作',
          type: 'template',
          template: 'opt'
        }
      ],
      // 添加分类对话框显示与隐藏
      addCateDialogVisible: false,
      // 添加分类表单
      addCateForm: {
        cat_name: '',
        cat_pid: 0,
        cat_level: 0
      },
      // 添加分类的规则 规则名称和属性名称一致
      addCateRules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' },
          { min: 3, max: 20, message: '请输入3-20个字符', trigger: 'blur' }
        ]
      },
      // 父级分类的数据
      parentCateList: [],
      cascaderProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      // 选择的父级分类id数组
      selectedKeys: [],
      // 编辑分类对话框显示与隐藏
      editCateDialogVisible: false
    }
  },

  created() {
    // 获取商品分类
    this.getCategories()
  },
  methods: {
    // 获取商品分类
    async getCategories() {
      const { data: res } = await this.$http.get('categories', {
        params: this.queryInfo
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类错误')
      }
      // this.$message.success('获取商品分类成功')
      this.cateList = res.data.result
      this.total = res.data.total
    },
    // 页面值发生改变 5条或10条
    handleSizeChange(newsize) {
      this.queryInfo.pagesize = newsize
      this.getCategories()
    },
    // 页面值 第几页
    handleCurrentChange(newnum) {
      this.queryInfo.pagenum = newnum
      this.getCategories()
    },
    // 显示添加分类对话框
    showAddCate() {
      // 获取父级分类
      this.getParentCateList()
      this.addCateDialogVisible = true
    },
    // 父级分类获取
    async getParentCateList() {
      const { data: res } = await this.$http.get('categories', { type: 2 })
      if (res.meta.status !== 200) {
        return this.$message.error('获取父级分类失败')
      }
      this.$message.success('获取父级分类成功')
      this.parentCateList = res.data
      // console.log(res.data)
    },
    // 父级分类   选择项发生改变事件
    parentCateChange() {
      if (this.selectedKeys.length > 0) {
        this.addCateForm.cat_pid = this.selectedKeys[
          this.selectedKeys.length - 1
        ]
        this.addCateForm.cat_level = this.selectedKeys.length
        return
      } else {
        this.addCateForm.cat_pid = 0
        this.addCateForm.cat_level = 0
      }
      console.log(this.selectedKeys)
    },
    // 关闭表单重置
    closeAddCateDialog() {
      this.$refs.addCateFormRef.resetFields()
      this.selectedKeys = []
      this.addCateForm.cat_pid = 0
      this.addCateForm.cat_level = 0
    },
    // 添加分类
    addCate() {
      // console.log(this.addCateForm)
      this.$refs.addCateFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post(
          'categories',
          this.addCateForm
        )
        if (res.meta.status !== 201) {
          return this.$message.error('添加分类失败')
        }
        this.$message.success('添加分类成功')
        this.getCategories()
        this.addCateDialogVisible = false
      })
    },
    // 显示编辑分类对话框
    async showEditCate(id) {
      this.getParentCateList()
      const { data: res } = await this.$http.get('categories/' + id)
      console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error('查询失败')
      }
      this.addCateForm = res.data
      this.editCateDialogVisible = true
    },
    // 编辑提交分类名称
    editCate() {
      this.$refs.addCateFormRef.validate(async valid => {
        if (!valid) return
        console.log(this.addCateForm)
        const { data: res } = await this.$http.put(
          `categories/${this.addCateForm.cat_id}`,
          {
            cat_name: this.addCateForm.cat_name
          }
        )
        if (res.meta.status !== 200) {
          return this.$message.error('修改分类名称失败')
        }
        // this.addCate()
        this.getCategories()
        this.$message.success('修改分类名称成功')
        this.editCateDialogVisible = false
      })
    },
    // 删除分类名称
    async delCate(id) {
      const confirmresult = await this.$confirm(
        '此操作将永久删除该分类名称, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)
      if (confirmresult !== 'confirm') {
        return this.$message.info('已取消了删除该分类名称')
      }
      const { data: res } = await this.$http.delete('categories/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('删除分类名称失败')
      }
      // this.queryInfo.pagenum = this.queryInfo.pagenum
      this.getCategories()

      this.$message.success('删除分类名称成功')
      if ((this.total - 1) % this.queryInfo.pagesize === 0) {
        this.queryInfo.pagenum = this.queryInfo.pagenum - 1
        this.getCategories()
      }
    }
  }
}
</script>

<style scoped lang="less">
.zk-table {
  margin: 15px 0;
}
.el-cascader {
  width: 100%;
}
.el-cascader {
  width: 100%;
  height: 100%;
  top: 0 !important;
}
</style>
