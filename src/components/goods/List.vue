<template>
  <div>
    <!-- 导航面包屑区域 -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path:'/home'}">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 内容区域 -->
    <el-card>
      <el-row :gutter="28">
        <el-col :span="8">
          <el-input placeholder="请输入内容" clearable @clear="getGoodsList" v-model="queryInfo.query">
            <el-button icon="el-icon-search" slot="append" @click="getGoodsList"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
            <el-button type="primary" @click="goAddGood">添加商品</el-button>
        </el-col>
      </el-row>
        <el-table :data="goodsList" style="width:100%" border>
          <el-table-column type="index"></el-table-column>
          <el-table-column prop="goods_name" label="商品名称" ></el-table-column>
          <el-table-column prop="goods_price" label="商品价格(元)" width="95px"></el-table-column>
          <el-table-column prop="goods_weight" label="商品重量" width="95px"></el-table-column>
          <el-table-column prop="add_time" label="创建时间" width="160px">
            <template v-slot="scope">{{scope.row.add_time | dateFormat }}</template>
          </el-table-column>
          <el-table-column label="操作" width="125px">
            <template v-slot="scope">
              <el-button type="primary" icon="el-icon-edit" size="mini"></el-button>
              <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeGood(scope.row.goods_id)"></el-button>
            </template>
          </el-table-column>
        </el-table>
        <el-pagination
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          :current-page="queryInfo.pagenum"
          :page-sizes="[10, 20, 30, 40]"
          :page-size="10"
          layout="total, sizes, prev, pager, next, jumper"
          :total="total">
        </el-pagination>
    </el-card>
  </div>
</template>
<script>
export default {
  data(){
    return {
      goodsList:[],
      queryInfo:{
        query:'',
        pagenum:1,
        pagesize:10
      },
      total:0
    }
  },
  created(){
    this.getGoodsList()
  },
  methods:{
    async getGoodsList(){
      const {data:res} = await this.$http.get('goods',{
      params: this.queryInfo})
      if(res.meta.status!==200) return this.$message.error('商品列表获取失败')
      this.$message.success('商品列表获取成功')
      this.goodsList = res.data.goods
      this.total = res.data.total
    },
    handleSizeChange(val){
      this.queryInfo.pagesize = val
      this.getGoodsList()
    },
    handleCurrentChange(val){
      this.queryInfo.pagenum = val
      this.getGoodsList()
    },
    async removeGood(goods_id){
      const confirmResult = await this.$confirm(
        '此操作将永久删除该商品, 是否继续?',
        '提示',
        {
          confirmButtonText:'确定',
          cancelButtonText:'取消',
          type:'warning'
        }
      ).catch(err=>err)
      if(confirmResult!=='confirm'){
        this.$message.info('已经取消删除')
      }
      const { data:res} = await this.$http.delete(`goods/${goods_id}`)
      if(res.meta.status!==200) return this.$message.error('删除商品失败')
      this.$message.success('删除商品成功')
      this.getGoodsList()
    },
    goAddGood(){
      this.$router.push('/goods/add')
    }
  }
}
</script>
<style lang="less" scoped>

</style>
