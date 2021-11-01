<template>
  <div>
    <!-- 面包屑导航区 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片试图 -->
    <el-card>
      <el-row>
        <el-col><el-button type="primary" @click="addDialogVisible = true">添加角色</el-button></el-col>
      </el-row>
      <!-- 角色列表区域 -->
      <el-table :data="roleList" border stripe>
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row :class="['bdbottom', index === 0 ? 'bdtop' : '', 'vcenter']" v-for="(item1, index) in scope.row.children" :key="item1.id">
              <!-- 渲染一级权限 -->
              <el-col :span="5">
                <el-tag closable @close="removeRightById(scope.row, item1.id)">{{item1.authName}}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 渲染二三级权限 -->
              <el-col :span="19">
                <el-row :class="[index2 === 0 ? '' : 'bdtop', 'vcenter']" v-for="(item2, index2) in item1.children" :key="item2.id">  
                  <el-col :span="6">
                    <el-tag closable @close="removeRightById(scope.row, item2.id)" type="success">{{item2.authName}}</el-tag>
                     <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="18">
                    <el-tag closable @close="removeRightById(scope.row, item3.id)" type="warning" v-for="(item3) in item2.children" :key="item3.id">
                      {{item3.authName}}
                    </el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作" width="300px">
          <template slot-scope="scope">
            <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.id)">编辑</el-button>
            <el-button type="danger" icon="el-icon-delete" size="mini" @click="deleteRole(scope.row)">删除</el-button>
            <el-button type="warning" icon="el-icon-setting" size="mini" @click="showSetRightDialog(scope.row)">分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!-- 添加角色对话框 -->
    <el-dialog
      title="添加角色"
      :visible.sync="addDialogVisible"
      width="50%"
      :close-on-click-modal="false"
      @close="addDialogClosed">
      <el-form :model="addForm" ref="addFormRef" label-width="70px">
        <el-form-item label="角色名称">
          <el-input v-model="addForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述">
          <el-input v-model="addForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addRole">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 编辑角色对话框 -->
    <el-dialog
      title="编辑角色"
      :visible.sync="editDialogVisible"
      width="50%"
      :close-on-click-modal="false"
      @close="editDialogClosed">
      <el-form :model="editForm" ref="editFormRef" label-width="70px">
        <el-form-item label="角色名称">
          <el-input v-model="editForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述">
          <el-input v-model="editForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editRole">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 分配角色权限对话框 -->
        <!-- 添加角色对话框 -->
    <el-dialog
      title="权限分配"
      :visible.sync="setRightDialog"
      width="50%"
      :close-on-click-modal="false"
      @close="setRightDialogClosed">
      <el-tree :data="rightsList" 
      :props="treeProps" 
      show-checkbox node-key="id"
      default-expand-all
      :default-checked-keys="defKeys"
      ref="treeRef"></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRightDialog = false">取 消</el-button>
        <el-button type="primary" @click="allotRights">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: 'Roles',
  data() {
    return {
      roleList: [],
      addForm: {
        roleName: '',
        roleDesc: ''
      },
      editForm: {},
      addDialogVisible: false,
      editDialogVisible: false,
      setRightDialog: false,
      rightsList: [],
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      defKeys: [],
      roleId: ''
    }
  },
  created() {
    this.getRolesList()
  },
  methods: {
    async getRolesList() {
      const {data} = await this.$http.get('roles')
      if(data.meta.status !== 200) return this.$message.error('获取角色列表失败!')
      this.roleList = data.data
    },
    addDialogClosed() {
      this.$refs.addFormRef.resetFields()
    },
    editDialogClosed() {
      this.$refs.editFormRef.resetFields()
    },
    async addRole() {
      const {data} = await this.$http.post('roles', this.addForm)
      console.log(data);
      if(data.meta.status !== 201) return this.$message.error('角色添加失败！')
      this.$message.success('角色添加成功！')
      this.addDialogVisible = false
      this.getRolesList()
    },
    async showEditDialog(id) {
      this.editDialogVisible = true
      const {data} = await this.$http.get(`roles/${id}`)
      if(data.meta.status !== 200) return this.$message.error('查询角色信息失败！')
      this.editForm = data.data
    },
    deleteRole(roleInfo) {
      this.$confirm(`确认要删除角色${roleInfo.roleName}吗？`, '提示', {
        confirmButtonText: '确认',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(async() => {
        const {data} = await this.$http.delete(`roles/${roleInfo.id}`)
        if(data.meta.status !== 200) return this.$message.error('角色删除失败！')
        this.$message({
          type: 'success',
          message: '角色删除成功！'
        })
        this.getRolesList()
      }).catch(() => {
        this.$message.info('删除操作取消！')
      })
    },
    async editRole() {
      const {data} = await this.$http.put(`roles/${this.editForm.roleId}`, {
        roleName: this.editForm.roleName,
        roleDesc: this.editForm.roleDesc
      })
      if(data.meta.status !== 200) return this.$message.error('角色信息修改失败！')
      this.$message.success('角色信息修改成功!')
      this.editDialogVisible = false
      this.getRolesList()
    },
    removeRightById(role, rightId) {
      this.$confirm(`确认要删除此权限吗？`, '提示', {
        confirmButtonText: '确认',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(async() => {
        const {data} = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
        if(data.meta.status !== 200) return this.$message.error('权限删除失败！')
        this.$message({
          type: 'success',
          message: '权限删除成功！'
        })
        role.children = data.data
      }).catch(() => {
        this.$message.info('删除操作取消！')
      })
    },
    async showSetRightDialog(role) {
      const {data} = await this.$http.get(`rights/tree`)
      if(data.meta.status !== 200) return this.$message.error('获取权限列表失败!')
      this.rightsList = data.data
      this.getLeafKeys(role, this.defKeys)
      this.roleId = role.id
      this.setRightDialog = true
    },
    getLeafKeys(node, arr) {
      if(!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item => {
        this.getLeafKeys(item,arr)
      })
    },
    setRightDialogClosed() {
      this.defKeys = []
    },
    async allotRights() {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(), 
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      const idStr = keys.join(',')
      const {data} = await this.$http.post(`roles/${this.roleId}/rights`, {rids: idStr})
      if(data.meta.status !== 200) return this.$message.error('权限更新失败！')
      this.$message.success('权限更新成功！')
      this.getRolesList()
      this.setRightDialog = false
    }
  }
}
</script>

<style lang="less" scoped>
.el-tag {
  margin: 7px 7px 7px 45px;
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