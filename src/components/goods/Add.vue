<template>
<div>
  <el-breadcrumb separator="/">
    <el-breadcrumb-item :to="{path:'/home'}">首页</el-breadcrumb-item>
    <el-breadcrumb-item>商品管理</el-breadcrumb-item>
    <el-breadcrumb-item>添加商品</el-breadcrumb-item>
  </el-breadcrumb>
  <el-card>
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
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px">
      <el-tabs :tab-position="'left'" v-model="activeIndex"
      :before-leave="beforeTabLeave" @tab-click="tabClicked">
        <el-tab-pane label="基本信息" name="0">
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
        </el-tab-pane>
        <el-tab-pane label="商品参数" name="1">
          <el-form-item :label="item.attr_name" :key="item.attr_id"
          v-for="item in manyTableData">
            <el-checkbox-group v-model="item.attr_vals">
              <el-checkbox border :label="val" v-for="(val,i) in item.attr_vals"
              :key="i"></el-checkbox>
            </el-checkbox-group>
          </el-form-item>
        </el-tab-pane>
        <el-tab-pane label="商品属性" name="2">
          <el-form-item :label="item.attr_name" :key="item.attr_id"
          v-for="item in onlyTableData">
            <el-input v-model="item.attr_vals"></el-input>
          </el-form-item>
        </el-tab-pane>
        <el-tab-pane label="商品图片" name="3">
          <!-- 商品图片上传 -->
          <el-upload :action="uploadURL" :on-preview="handlePreview" 
          :on-remove="handleRemove"
          :on-success="handleSuccess"
          list-style="picture"
          :headers="headerObj">
            <el-button size="small" type="primary">点击上传</el-button>
          </el-upload>
        </el-tab-pane>
        <!-- 预览图片对话框 -->
        <el-dialog title="图片预览" :visible.sync="previewVisible" width="50%">
          <img :src="previewPath" class="previewImg"/>
        </el-dialog>
        <el-tab-pane label="商品内容" name="4">
          <!-- 富文本编辑器组件 -->
          <div class="ql-snow">
          <quill-editor v-model="addForm.goods_introduce" class="ql-editor"></quill-editor>
          </div>
          <el-button type="primary" class="btnAdd" @click="add">添加商品</el-button>
        </el-tab-pane>
      </el-tabs>
    </el-form>
  </el-card>
</div>
</template>
<script>
import _ from 'lodash'
export default {
  data(){
    return {
      activeIndex:'0',
      addForm:{
        goods_name:'',
        goods_price:0,
        goods_weight:0,
        goods_number:0,
        goods_cat:[],
        //上传图片数组
        pics:[],
        goods_introduce:'',
        attrs:[]
      },
      //上传图片的url地址
      uploadURL:'http://127.0.0.1:8888/api/private/v1/upload',
      //图片上传组件的headers请求头对象
      headerObj:{Authorization:window.sessionStorage.getItem('token')},
      //保存预览图片的url地址
      previewPath:'',
      //控制预览图片对话框的显示和隐藏
      previewVisible:false,
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
      },
      manyTableData:[],
      onlyTableData:[],
     
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
      console.log(this.addForm)
      if(oldActiveName === '0'){
        if(this.addForm.goods_cat.length !== 3){
          this.$message.error('请选择商品分类')
          return false
        }else if(this.addForm.goods_name.trim()===''){
          this.$message.error('请输入商品名称')
          return false
        }else if(this.addForm.goods_price=='0'){
          this.$message.error('请输入商品价格')
          return false
        }else if(this.addForm.goods_weight.trim()===''){
          this.$message.error('请输入商品重量')
          return false
        }else if(this.addForm.goods_number==='0'){
          this.$message.error('请输入商品数量')
          return false
        }  
      }
    },
    async tabClicked(){
      if (this.activeIndex === '1'){
        const {data:res} = await this.$http.get(
        `categories/${this.cateId}/attributes`,
        {params:{sel:'many'}}
        )
        if(res.meta.status !== 200){
          return this.$message.error('获取动态参数列表失败')
        }
        res.data.forEach(item =>{
          item.attr_vals = item.attr_vals.length===0?[]
          : item.attr_vals.split(' ')
        })
        this.manyTableData = res.data
      }else if(this.activeIndex==='2'){
        const {data:res} = await this.$http.get(
        `categories/${this.cateId}/attributes`,
        {params:{sel:'only'}}
        )
        if(res.meta.status !== 200){
          return this.$message.error('获取静态属性列表失败')
        }
        this.onlyTableData = res.data
      }
    },
    handlePreview(file){
      //图片预览时执行
      this.previewPath = file.response.data.url
      this.previewVisible = true
    },
    handleRemove(file){
      // 获取要删除图片的临时路径
      const filePath = file.response.data.tmp_path
      // 查找符合条件的索引
      const index = this.addForm.pics.findIndex(item=>item.pics
      === filePath)
      // 移除索引对应的图片
      this.addForm.pics.splice(index,1)
    },
    handleSuccess(response) {
      // 将服务器返回的临时路径保存到addForm表单的pics数组中
      this.addForm.pics.push({
        pic:response.data.tmp_path
      })
    },
    add(){
      this.$refs.addFormRef.validate(async valid => {
        if(!valid) return this.$message.error('请填写必要的表单项!')
        //将 addForm进行深拷贝，避免goods_cat数组转换字符串之后导致级联选择器报错
        const form = _.cloneDeep(this.addForm)
        form.goods_cat = form.goods_cat.join(',')
        this.manyTableData.forEach(item=>{
          form.attrs.push({attr_id:item.attr_id,
          attr_value:item.attr_vals.join(" ")})
        })
        this.onlyTableData.forEach(item=>{
          form.attrs.push({attr_id:item.attr_id,
          attr_value:item.attr_vals})
        })
        const {data:res} = await this.$http.post('goods',form)
        if(res.meta.status!==201){
          return this.$message.error('添加商品失败')
        }
        this.$message.success('添加商品成功')
        this.$router.push('/goods')
      })
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
.previewImg {
  width: 100%;
}

</style>
