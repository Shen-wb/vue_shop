<template>
  <div>
    <!-- 面包屑导航区 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品品类</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片试图区域 -->
    <el-card>
      <el-row>
        <el-col><el-button type="primary" @click="showAddCateDialog">添加分类</el-button></el-col>
      </el-row>
      <!-- 表格区域 -->
      <tree-table :data="cateList" :columns="columns" :selection-type="false" :expand-type="false" show-index index-text="#" border :show-row-hover="false">
        <template slot="isOk" slot-scope="scope">
          <i class="el-icon-success" v-if="scope.row.cat_deleted === false" style="color: green"></i>
          <i class="el-icon-error" v-else style="color: red"></i>
        </template>
        <template slot="order" slot-scope="scope">
          <el-tag size="mini" v-if="scope.row.cat_level === 0">一级</el-tag>
          <el-tag type="success" size="mini" v-else-if="scope.row.cat_level === 1">二级</el-tag>
          <el-tag type="warning" size="mini" v-else>三级</el-tag>
        </template>
        <template slot="opt" slot-scope="scope">
          <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditCate(scope.row)">编辑</el-button>
          <el-button type="danger" icon="el-icon-delete" size="mini" @click="deleteCate(scope.row)">删除</el-button>
        </template>
      </tree-table>
      <!-- 分页区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[3, 5, 10, 15]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total">
      </el-pagination>
    </el-card>
    <!-- 添加分类对话框 -->
    <el-dialog
      title="添加分类"
      :visible.sync="addDialogVisible"
      :close-on-click-modal="false"
      width="50%"
      @close="addCateDialogClosed">
      <el-form :model="addCateForm" :rules="addCateFormRules" ref="addCateFormRef" label-width="100px">
        <el-form-item label="分类名称" prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类">
           <el-cascader v-model="selectedKeys" :options="parentCateList" @change="handleChange" :props="cascaderProps" clearable></el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改分类对话框 -->
     <el-dialog
      title="修改分类"
      :visible.sync="editDialogVisible"
      :close-on-click-modal="false"
      width="50%"
      @close="editCateDialogClosed">
      <el-form :model="editCateForm" :rules="editCateFormRules" ref="editCateFormRef" label-width="100px">
        <el-form-item label="分类名称" prop="cat_name">
          <el-input v-model="editCateForm.cat_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
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
      queryInfo: {type: 3, pagenum: 1, pagesize: 5},
      cateList: [],
      total: 0,
      addDialogVisible: false,
      editDialogVisible: false,
      addCateForm: {
        cat_name: '',
        cat_pid: 0,
        cat_level: 0
      },
      editCateForm: {cat_name: '', cat_id: ''},
      addCateFormRules: {
        cat_name: [
          {required: true, message: '请输入分类名称', trigger: 'blur'}
        ]
      },
      editCateFormRules: {
        cat_name: [
          {required: true, message: '请输入分类名称', trigger: 'blur'}
        ]
      },
      columns: [
        {
          label: '分类名称',
          prop: 'cat_name',
        },
        {
          label: '是否有效',
          type: 'template',
          template: 'isOk',
        },
        {
          label: '排序',
          type: 'template',
          template: 'order',
        },
        {
          label: '操作',
          type: 'template',
          template: 'opt',
        },
      ],
      parentCateList: [],
      cascaderProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children',
        checkStrictly: true,
        expandTrigger: 'hover'
      },
      selectedKeys: [],
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    async getCateList() {
      const {data} = await this.$http.get('categories', {params: this.queryInfo})
      if(data.meta.status !== 200) return this.$message.error('获取品类列表失败!')
      this.cateList = data.data.result
      this.total = data.data.total
    },
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getCateList()
    },
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getCateList()
    },
    addCate() {
      this.$refs.addCateFormRef.validate(async valid => {
        if(!valid) return
        const {data} = await this.$http.post('categories', this.addCateForm)
        if(data.meta.status !== 201) return this.$message.error('分类添加失败!')
        this.$message.success('分类添加成功!')
        this.getCateList()
        this.addDialogVisible = false
      })
    },
    showAddCateDialog() {
      this.addDialogVisible = true
      this.getParentCateList()
    },
    async getParentCateList() {
      const {data} = await this.$http.get('categories', {params: {type: 2}})
      if(data.meta.status !== 200) return this.$message.error('获取父级分类数据失败!')
      this.parentCateList = data.data
    },
    handleChange() {
      if(this.selectedKeys.length > 0) {
        this.addCateForm.cat_pid = this.selectedKeys[this.selectedKeys.length - 1]
        this.addCateForm.cat_level = this.selectedKeys.length
      } else {
        this.addCateForm.cat_pid = 0
        this.addCateForm.cat_level = 0
      }
    },
    addCateDialogClosed() {
      this.$refs.addCateFormRef.resetFields()
      this.addCateForm.cat_pid = 0
      this.addCateForm.cat_level = 0
      this.selectedKeys = []
    },
    editCateDialogClosed() {
      this.$refs.editCateFormRef.resetFields()
    },
    deleteCate(cateInfo) {
      this.$confirm(`确认要删除${cateInfo.cat_name}分类吗？`, '提示', {
        confirmButtonText: '确认',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(async() => {
        const {data} = await this.$http.delete(`categories/${cateInfo.cat_id}`)
        if(data.meta.status !== 200) return this.$message.error('分类删除失败！')
        console.log(data);
        this.$message({
          type: 'success',
          message: '分类删除成功！'
        })
        console.log(this.queryInfo);
        this.getCateList()
      }).catch(() => {
        this.$message.info('删除操作取消！')
      })
    },
    showEditCate(cateInfo) {
      this.editCateForm.cat_name = cateInfo.cat_name
      this.editCateForm.cat_id = cateInfo.cat_id
      this.editDialogVisible = true
    },
    editCate() {
      this.$refs.editCateFormRef.validate(async valid => {
        if(!valid) return
        const {data} = await this.$http.put(`categories/${this.editCateForm.cat_id}`, {cat_name: this.editCateForm.cat_name})
        if(data.meta.status !== 200) return this.$message.error('分类修改失败!')
        this.$message.success('分类修改成功!')
        this.getCateList()
        this.editDialogVisible = false
      })
    }
  },

}
</script>

<style lang="less" scoped>
.el-cascader {
  width: 100%;
}
</style>