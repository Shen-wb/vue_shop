<template>
  <div>
    <!-- 面包屑导航区 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片试图区域 -->
    <el-card>
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="getGoodsList" @keyup.enter.native="getGoodsList">
            <el-button slot="append" icon="el-icon-search" @click="getGoodsList" ></el-button>
          </el-input>
        </el-col>
        <el-col :span="4"><el-button type="primary" @click="toAddGoodsPage">添加商品</el-button></el-col>
      </el-row>
      <el-table :data="goodsList" border stripe> 
        <el-table-column type="index"></el-table-column>
        <el-table-column label="商品名称" prop="goods_name"></el-table-column>
        <el-table-column label="商品价格(元)" prop="goods_price" width="95px"></el-table-column>
        <el-table-column label="商品重量" prop="goods_weight" width="70px"></el-table-column>
        <el-table-column label="创建时间" width="140px">
          <template slot-scope="scope">{{scope.row.add_time | timeFormat}}</template>
        </el-table-column>
        <el-table-column label="操作" width="130px">
          <template slot-scope="scope">
            <el-button type="primary" icon="el-icon-s-fold" size="mini" @click="showEditDialog(scope.row.goods_id)"></el-button>
            <el-button type="danger" icon="el-icon-delete" size="mini" @click="deleteById(scope.row)"></el-button>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[5,10,15,20]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total" background>
      </el-pagination>
    </el-card>
    <!-- 编辑商品信息对话框 -->
    <el-dialog
      title="编辑商品"
      :visible.sync="editDialogVisible"
      :close-on-click-modal="false"
      width="50%">
      <el-form :model="goodsInfo" ref="editFormRef" label-width="100px">
        <el-form-item label="商品名称：">
          <span>{{goodsInfo.goods_name}}</span>
        </el-form-item>
        <el-form-item label="商品价格：">
          <span>{{goodsInfo.goods_price}}元</span>
        </el-form-item>
        <el-form-item label="商品重量：">
          <span>{{goodsInfo.goods_weight}}克</span>
        </el-form-item>
        <el-form-item label="商品数量：">
          <span>{{goodsInfo.goods_number}}</span>
        </el-form-item>
        <el-form-item label="商品参数：">
          <el-tag v-for="(attr, index) in goodsInfo.attrs" :key="index">{{attr.attr_vals}}</el-tag>
        </el-form-item>
        <label style="margin-left: 18px">商品介绍：</label>
        <div v-html="goodsInfo.goods_introduce"></div>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editDialogVisible = false">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import dayjs from 'dayjs'
export default {
  name: 'List',
  data() {
    return {
      queryInfo: {
        query: '',
        pagenum: 1, 
        pagesize: 10
      },
      goodsList: [],
      total: 0,
      editDialogVisible: false,
      goodsInfo: {},
    }
  },
  created() {
    this.getGoodsList()
  },
  methods: {
    async getGoodsList() {
      const {data} = await this.$http.get('goods', {params: this.queryInfo})
      if(data.meta.status !== 200) return this.$message.error('获取商品列表失败!')
      this.goodsList = data.data.goods
      this.total = data.data.total
    },
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getGoodsList()
    },
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getGoodsList()
    },
    deleteById(goodsInfo) {
      this.$confirm(`确认要删除商品${goodsInfo.goods_name}吗？`, '提示', {
        confirmButtonText: '确认',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(async() => {
        const {data} = await this.$http.delete(`goods/${goodsInfo.goods_id}`)
        if(data.meta.status !== 200) return this.$message.error('商品删除失败！')
        this.$message({
          type: 'success',
          message: '商品删除成功！'
        })
        this.getGoodsList()
      }).catch(() => {
        this.$message.info('删除操作取消！')
      })
    },
    toAddGoodsPage() {
      this.$router.push('/goods/add')
    },
    async showEditDialog(id) {
      this.editDialogVisible = true
      const {data} = await this.$http.get(`goods/${id}`)
      if(data.meta.status !== 200) return this.$message.error('获取商品列表失败!')
      this.goodsInfo = data.data
      console.log(this.goodsInfo.attrs);
    }
  },
  filters: {
    timeFormat(time) {
      return dayjs(time).format('YYYY-MM-DD HH:mm:ss')
    }
  }
}
</script>

<style lang="less" scoped>
.el-tag {
  margin-right: 15px;
}
</style>