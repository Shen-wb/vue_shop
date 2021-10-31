<template>
  <div>
    <!-- 面包屑导航区 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card>
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input placeholder="请输入搜索关键词" v-model="queryInfo.query" clearable @clear="getUserList" @input="handleChange" @keyup.enter.native="getUserList">
            <el-button slot="append" icon="el-icon-search" @click="getUserList"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="addDialogVisible = true">添加用户</el-button>
        </el-col>
      </el-row>
      <!-- 用户列表区域 -->
      <el-table :data="userList" border stripe>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="姓名" prop="username"></el-table-column>
        <el-table-column label="邮箱" prop="email"></el-table-column>
        <el-table-column label="电话" prop="mobile"></el-table-column>
        <el-table-column label="角色" prop="role_name"></el-table-column>
        <el-table-column label="状态">
          <!-- 通过scope.row就可以获得这一行的所有数据 -->
          <template slot-scope="scope"> 
            <el-switch v-model="scope.row.mg_state" @change="userChangeState(scope.row)"></el-switch>
          </template>
        </el-table-column>
        <el-table-column label="操作" width="180px">
          <template slot-scope="scope">
            <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.id)"></el-button>
            <el-button type="danger" icon="el-icon-delete" size="mini" @click="deleteUser(scope.row)"></el-button>
            <el-tooltip effect="dark" content="分配角色" placement="top" :enterable="false">
              <el-button type="warning" icon="el-icon-setting" size="mini" ></el-button>
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
        :page-size="queryInfo.paegsize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total">
      </el-pagination>
    </el-card>
    <!-- 添加用户对话框 -->
    <el-dialog
      title="添加用户"
      :visible.sync="addDialogVisible"
      width="50%"
      :close-on-click-modal="false"
      @close="addDialogClosed">
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="70px">
        <el-form-item label="用户名" prop="username">
          <el-input v-model="addForm.username"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input v-model="addForm.password"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="addForm.email"></el-input>
        </el-form-item>
        <el-form-item label="手机号" prop="mobile">
          <el-input v-model="addForm.mobile"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addUser">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改用户对话框 -->
    <el-dialog
      title="修改用户"
      :visible.sync="editDialogVisible"
      width="50%"
      :close-on-click-modal="false"
       @close="editDialogClosed">
      <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="70px">
        <el-form-item label="用户名">
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
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editUser">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: 'User',
  data() {
    let checkEmail = (rule, value, cb) => {
      const regEmail = /^[a-zA-Z0-9]+([-_.][a-zA-Z0-9]+)*@[a-zA-Z0-9]+([-_.][a-zA-Z0-9]+)*\.[a-z]{2,}$/
      if(regEmail.test(value)) {
        return cb()
      }
      cb(new Error('请输入合法邮箱！'))
    }
    let checkMobile = (rule, value, cb) => {
      const regMobile = /^(0|86|17951)?(13[0-9]|15[0123456789]|17[678]|18[0-9]|14[57])[0-9]{8}$/
      if(regMobile.test(value)) {
        return cb()
      }
      cb(new Error('请输入合法手机号！'))
    }
    return {
      queryInfo: {
        query: '', 
        pagenum: 1,
        pagesize: 10
      },
      userList: [],
      total: 0,
      addDialogVisible: false,
      editDialogVisible: false,
      addForm: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      editForm: {},
      addFormRules: {
        username: [
          {required: true, message: '请输入用户名!', rigger: 'blur'},
          {min: 3, max: 10, message: '用户名长度在3~10之间!', rigger: 'blur'}
        ],
        password: [
          {required: true, message: '请输入密码!', rigger: 'blur'},
          {min: 6, max: 15, message: '密码长度在6~15之间!', rigger: 'blur'}
        ],
        email: [
          {required: true, message: '请输入邮箱!', rigger: 'blur'},
          {validator: checkEmail, trigger: 'blur'}
        ],
        mobile: [
          {required: true, message: '请输入手机号', rigger: 'blur'},
          {validator: checkMobile, trigger: 'blur'}
        ]
      },
      editFormRules: {
        email: [
          {required: true, message: '请输入邮箱!', rigger: 'blur'},
          {validator: checkEmail, trigger: 'blur'}
        ],
        mobile: [
          {required: true, message: '请输入手机号', rigger: 'blur'},
          {validator: checkMobile, trigger: 'blur'}
        ]
      }
    }
  },
  created() {
    this.getUserList()
  },
  methods: {
    async getUserList() {
      const {data} = await this.$http.get('users', {
        params: this.queryInfo
      })
      if(data.meta.status !== 200) return this.$message.error('获取用户列表失败!')
      this.userList = data.data.users
      this.total = data.data.total
    },
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getUserList()
    },
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getUserList()
    },
    async userChangeState(userInfo) {
      const {data} = await this.$http.put(`users/${userInfo.id}/state/${userInfo.mg_state}`)
      if(data.meta.status !== 200) {
        userInfo.mg_state = !userInfo.mg_state
        return this.$message.error('用户状态更新失败!')
      }
      this.$message.success('用户状态更新成功!')
    },
    handleChange() {
      if(this.queryInfo.query === '') {
        this.getUserList()
      }
    },
    addUser() {
      this.$refs.addFormRef.validate(async(valid) => {
        if(!valid) return
        const {data} = await this.$http.post('users',this.addForm)
        if(data.meta.status !== 201) return this.$message.error('用户添加失败！')
        this.$message.success('用户添加成功!')
        this.addDialogVisible = false
        this.getUserList()
      })
    },
    addDialogClosed() {
      this.$refs.addFormRef.resetFields()
    },
    editDialogClosed() {
      this.$refs.editFormRef.resetFields()
    },
    async showEditDialog(id) {
      this.editDialogVisible = true
      const {data} = await this.$http.get(`users/${id}`)
      if(data.meta.status !== 200) return this.$message.error('查询用户信息失败！')
      this.editForm = data.data
    },
    editUser() {
      this.$refs.editFormRef.validate(async(valid) => {
        if(!valid) return
        const {data} = await this.$http.put(`users/${this.editForm.id}`, {
          email: this.editForm.email,
          mobile: this.editForm.mobile
        })
        if(data.meta.status !== 200) return this.$message.error('用户信息修改失败！')
        this.$message.success('用户信息修改成功!')
        this.editDialogVisible = false
        this.getUserList()
      })
    },
    deleteUser(userInfo) {
      this.$confirm(`确认要删除用户${userInfo.username}吗？`, '提示', {
        confirmButtonText: '确认',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(async() => {
        const {data} = await this.$http.delete(`users/${userInfo.id}`)
        if(data.meta.status !== 200) return this.$message.error('用户删除失败！')
        this.$message({
          type: 'success',
          message: '用户删除成功！'
        })
        this.getUserList()
      }).catch(() => {
        this.$message.info('删除操作取消！')
      })
    }
  }
}
</script>

<style lang="less" scoped>

</style>