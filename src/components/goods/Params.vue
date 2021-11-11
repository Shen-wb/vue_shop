<template>
  <div>
    <!-- 面包屑导航区 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>分类参数</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片试图区域 -->
    <el-card>
       <el-alert title="注意：只允许为第三级分类设置相关参数" type="warning" show-icon :closable="false"></el-alert>
       <el-row class="cat_opt">
         <el-col>
            <span>选择商品分类：</span>
            <el-cascader expand-trigger="hover" v-model="selectedCateKeys" :options="cateList" :props="cateProps" @change="handleChange"></el-cascader>
         </el-col>
       </el-row>
       <el-tabs v-model="activeName" @tab-click="handleClick">
        <el-tab-pane label="动态参数" name="many">
          <el-button type="primary" size="mini" :disabled="isBtnDisabled" @click="showAddDialog">添加参数</el-button>
          <el-table :data="manyTableData" border stripe>
            <el-table-column type="expand">
              <template slot-scope="scope">
                <el-tag closable v-for="(item, index) in scope.row.attr_vals" :key="index" @close="handleClose(index, scope.row)">{{item}}</el-tag>
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)"></el-input>
                <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
              </template>
            </el-table-column>
            <el-table-column type="index"></el-table-column>
            <el-table-column label="参数名称" prop="attr_name"></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row)">编辑</el-button>
                <el-button type="danger" icon="el-icon-delete" size="mini" @click="deleteParams(scope.row)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
        <el-tab-pane label="静态属性" name="only">
           <el-button type="primary" size="mini" :disabled="isBtnDisabled" @click="showAddDialog">添加属性</el-button>
            <el-table :data="onlyTableData" border stripe>
              <el-table-column type="expand">
                <template slot-scope="scope">
                  <el-tag closable v-for="(item, index) in scope.row.attr_vals" :key="index" @close="handleClose(index, scope.row)">{{item}}</el-tag>
                  <el-input
                    class="input-new-tag"
                    v-if="scope.row.inputVisible"
                    v-model="scope.row.inputValue"
                    ref="saveTagInput"
                    size="small"
                    @keyup.enter.native="handleInputConfirm(scope.row)"
                    @blur="handleInputConfirm(scope.row)"></el-input>
                  <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
                </template>
              </el-table-column>
              <el-table-column type="index"></el-table-column>
              <el-table-column label="属性名称" prop="attr_name"></el-table-column>
              <el-table-column label="操作">
                <template slot-scope="scope">
                  <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row)">编辑</el-button>
                  <el-button type="danger" icon="el-icon-delete" size="mini" @click="deleteParams(scope.row)">删除</el-button>
                </template>
              </el-table-column>
            </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>
    <!-- 添加参数属性对话框 -->
    <el-dialog
      :title="'添加' + titleText"
      :visible.sync="addDialogVisible"
      width="50%"
      :close-on-click-modal="false"
      @close="addDialogClosed">
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px">
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="addForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addParams">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改参数属性对话框 -->
    <el-dialog
      :title="'修改' + titleText"
      :visible.sync="editDialogVisible"
      width="50%"
      :close-on-click-modal="false"
      @close="editDialogClosed">
      <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="100px">
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="editForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editParams">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: 'Params',
  data() {
    return {
      cateList: [],
      cateProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children',
        expandTrigger: 'hover'
      },
      selectedCateKeys: [],
      activeName: 'many',
      manyTableData: [],
      onlyTableData: [],
      addDialogVisible: false,
      editDialogVisible: false,
      addForm: {
        attr_name: ''
      },
      editForm: {},
      addFormRules: {
        attr_name: [{required: true, message: `请输入参数名称`, trigger: 'blur'}]
      },
      editFormRules: {
        attr_name: [{required: true, message: `请输入参数名称`, trigger: 'blur'}]
      }
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    async getCateList() {
      const {data} = await this.$http.get('categories')
      if(data.meta.status !== 200) return this.$message.error('获取品类列表失败!')
      this.cateList = data.data
    },
    async getParamsData() {
      if(this.selectedCateKeys.length !== 3) {
        this.manyTableData = []
        this.onlyTableData = []
        this.selectedCateKeys = []
        return
      }
      const {data} = await this.$http.get(`categories/${this.cateId}/attributes`, {params:{sel: this.activeName}})
      if(data.meta.status !== 200) return this.$message.error('获取参数列表失败!')
      data.data.forEach(item => {
        item.attr_vals = item.attr_vals ? item.attr_vals.split(",") : []
        item.inputVisible = false
        item.inputValue = ''
      })
      if(this.activeName === 'many') {
        this.manyTableData = data.data
      } else {
        this.onlyTableData = data.data
      }
    },
    handleChange() {
      this.getParamsData()
      
    },
    handleClick() {
      this.getParamsData()
    },
    showAddDialog() {
      this.addDialogVisible = true
    },
    addDialogClosed() {
      this.$refs.addFormRef.resetFields()
    },
    editDialogClosed() {
      this.$refs.editFormRef.resetFields()
    },
    addParams() {
      this.$refs.addFormRef.validate(async valid => {
        if(!valid) return
        const {data} = await this.$http.post(`categories/${this.cateId}/attributes`,{
          attr_name: this.addForm.attr_name,
          attr_sel: this.activeName
        })
        if(data.meta.status !== 201) return this.$message.error(this.titleText+"添加失败!")
        this.$message.success(this.titleText+"添加成功!")
        this.addDialogVisible = false
        this.getParamsData()
      })
    },
    async showEditDialog(paramsInfo) {
      this.editDialogVisible = true
      const {data} = await this.$http.get(`categories/${this.cateId}/attributes/${paramsInfo.attr_id}`, {attr_sel: this.activeName})
      if(data.meta.status !== 200) return this.$message.error('获取参数名称失败!')
      this.editForm = data.data
    },
    editParams() {
      this.$refs.editFormRef.validate(async valid => {
        if(!valid) return
        const {data} = await this.$http.put(`categories/${this.cateId}/attributes/${this.editForm.attr_id}`, {
          attr_name: this.editForm.attr_name,
          attr_sel: this.activeName
        })
        if(data.meta.status !== 200) return this.$message.error('修改参数失败!')
        this.$message.success('修改参数成功!')
        this.getParamsData()
        this.editDialogVisible = false
      })
    },
    deleteParams(paramsInfo) {
      this.$confirm(`确认要删除${paramsInfo.attr_name}参数吗？`, '提示', {
        confirmButtonText: '确认',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(async() => {
        const {data} = await this.$http.delete(`categories/${this.cateId}/attributes/${paramsInfo.attr_id}`)
        if(data.meta.status !== 200) return this.$message.error(`${this.titleText}删除失败！`)
        console.log(data);
        this.$message({
          type: 'success',
          message: `${this.titleText}删除成功！`
        })
        this.getParamsData()
      }).catch(() => {
        this.$message.info('删除操作取消！')
      })
    },
    handleInputConfirm(row) {
      if(row.inputValue.trim().length === 0) {
        row.inputValue = ''
        row.inputVisible = false
        return
      }
      row.attr_vals.push(row.inputValue.trim())
      row.inputValue = ''
      row.inputVisible = false
      this.saveAttrVals(row)
    },
    async saveAttrVals(row) {
      const {data} = await this.$http.put(`categories/${this.cateId}/attributes/${row.attr_id}`, {
        attr_name: row.attr_name,
        attr_sel: row.attr_sel,
        attr_vals: row.attr_vals.join(',')
      })
      if(data.meta.status !== 200) return this.$message.error('修改参数项失败!')
      this.$message.success('修改参数项成功!')
    },
    showInput(row) {
      row.inputVisible = true
      this.$nextTick(_=> {
        this.$refs.saveTagInput.$refs.input.focus()
      })
    },
    handleClose(index, row) {
      row.attr_vals.splice(index, 1)
      this.saveAttrVals(row)
    }
  },
  computed: {
    isBtnDisabled() {
      if(this.selectedCateKeys.length !== 3) return true
      return false
    },
    cateId() {
      if(this.selectedCateKeys.length === 3) return this.selectedCateKeys[2]
      return null
    },
    titleText() {
      if(this.activeName === 'many') return '动态参数'
      return '静态属性'
    }
  }
}
</script>

<style lang="less" scoped>
.cat_opt {
  margin: 15px 0;
}
.el-tag {
  margin: 10px;
}
.input-new-tag {
  width: 120px;
  margin-left: 10px;
}
.button-new-tag {
  margin-left: 10px;
}
</style>