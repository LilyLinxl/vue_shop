<template>
<div>
  <el-breadcrumb separator="/">
    <el-breadcrumb-item :to="{path:'/home'}">首页</el-breadcrumb-item>
    <el-breadcrumb-item>商品管理</el-breadcrumb-item>
    <el-breadcrumb-item>添加商品</el-breadcrumb-item>
  </el-breadcrumb>
  <el-card>
    <el-row>
      <el-alert type="info" title="添加商品信息" show-icon center></el-alert>
      <el-steps :space="200" :active="activeIndex-0" finish-status="success"
      align-center>
        <el-step title="基本信息"></el-step>
        <el-step title="商品参数"></el-step>
        <el-step title="商品属性"></el-step>
        <el-step title="商品图片"></el-step>
        <el-step title="商品内容"></el-step>
        <el-step title="完成"></el-step>
      </el-steps>
    </el-row>
    <el-row>
      <el-tabs :tab-position="'left'" v-model="activeIndex"
      :before-leave="beforeTabLeave">
        <el-tab-pane label="基本信息">
          <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px">
            <el-form-item label="商品名称" prop="goods_name">
              <el-input v-model="addForm.goods_name" placeholder="请输入商品名称"></el-input>
            </el-form-item>
            <el-form-item label="商品价格" prop="goods_price">
              <el-input v-model="addForm.goods_price" type="number"></el-input>
            </el-form-item>
            <el-form-item label="商品重量" prop="goods_weight">
              <el-input v-model="addForm.goods_weight" type="number"></el-input>
            </el-form-item>
            <el-form-item label="商品数量" prop="goods_number">
              <el-input v-model="addForm.goods_number" type="number"></el-input>
            </el-form-item>
            <el-form-item label="商品分类" prop="goods_cat">
              <el-cascader  :options="catelist" :props="cateProps"
              v-model="addForm.goods_cat" @change="handleChange"></el-cascader>
            </el-form-item>
          </el-form>
        </el-tab-pane>
        <el-tab-pane label="商品参数" name="1">商品参数</el-tab-pane>
        <el-tab-pane label="商品属性" name="2">商品属性</el-tab-pane>
        <el-tab-pane label="商品图片" name="3">商品图片</el-tab-pane>
        <el-tab-pane label="商品内容" name="4">商品内容</el-tab-pane>
      </el-tabs>
    </el-row>
  </el-card>
</div>
</template>
<script>
export default {
  data(){
    return {
      activeIndex:'0',
      addForm:{
        goods_name:'',
        goods_price:0,
        goods_weight:0,
        goods_number:0,
        goods_cat:[]
      },
      addFormRules:{
        goods_name:[
          {required:true,message:'请输入商品名称',trigger:'blur'}
        ],
        goods_price:[
          {required:true,message:'请输入商品价格',trigger:'blur'}
        ],
        goods_weight:[
          {required:true,message:'请输入商品重量',trigger:'blur'}
        ],
        goods_number:[
          {required:true,message:'请输入商品数量',trigger:'blur'}
        ],
        goods_cat:[
          {required:true,message:'请选择商品分类',trigger:'blur'}
        ]
      },
      catelist:[],
      cateProps:{
        label:'cat_name',
        value:'cat_id',
        children:'children',
        expandTrigger:"hover"
      }
    }
  },
  created(){
    this.getCateList()
  },
  methods:{
    async getCateList(){
      const {data:res} = await this.$http.get('categories')
      if(res.meta.status!==200) return this.$message.error('')
      this.catelist = res.data
    },
    handleChange(){
      if(this.addForm.goods_cat.length !== 3){
        this.addForm.goods_cat = []
        return
      }
    },
    beforeTabLeave(activeName,oldActiveName){
      if(oldActiveName === '0'){
        if(this.addForm.goods_cat.length !== 3){
          this.$message.error('请选择商品分类')
          return false
        }else if(this.addForm.goods_name.trim()===''){
          this.$message.error('请输入商品名称')
          return false
        }else if(this.addForm.goods_price.trim()==='0'){
          this.$message.error('请输入商品价格')
          return false
        }else if(this.addForm.goods_weight.trim()===''){
          this.$message.error('请输入商品重量')
          return false
        }else if(this.addForm.goods_number.trim()==='0'){
          this.$message.error('请输入商品数量')
          return false
        }  
      }
    }
  },
  computed:{
    cateId(){
      if (this.addForm.goods_cat.length === 3) {
        return this.addForm.goods_cat[2]
      }
      return null
    }
  }
}
</script>
<style lang="less" scoped>

</style>
