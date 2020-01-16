<template>
    <div>
        <h3>分类参数</h3>
        <!-- 面包屑导航 -->
        <el-breadcrumb separator="/"> 
        <el-breadcrumb-item :to="{path:'/home'}">首页</el-breadcrumb-item>
          <el-breadcrumb-item>商品管理</el-breadcrumb-item>
          <el-breadcrumb-item>分类参数</el-breadcrumb-item>
        </el-breadcrumb>
        <!-- 卡片视图区域 -->
        <el-card>
          <!-- 警告区域 -->
          <el-alert title="注意:只允许为第三级分类设置相关参数" type="warning"
          :closeable="false" show-icon></el-alert>
          <!-- 选择商品分类区域 -->
          <el-row class="cat_opt">
            <el-col>
              <span>选择商品分类：</span>
              <!-- 选择商品分类的级联选择框 -->
              <el-cascader expandTrigger="hover" v-model="selectedCateKeys"
              :options="cateList" :props="cateProps" @change="handleChange"
              clearable></el-cascader>
            </el-col>
            <el-col></el-col>
          </el-row>
        </el-card>
         <el-tabs v-model="activeName" @tab-click="handleTabClick">
           <!-- 添加动态参数的面板 将标签页改为many-->
          <el-tab-pane label="动态参数" name="many">
            <el-button size="mini" type="primary" :disabled="isButtonDisabled">添加参数</el-button>
            <!-- 动态参数表格 -->
            <el-table :data="manyTableData" border stripe>
              <!-- 展开行 -->
              <el-table-column type="expand"></el-table-column>
              <el-table-column type="index"></el-table-column>
              <el-table-column label="参数名称" prop="attr_name"></el-table-column>
              <el-table-column label="操作">
                <template v-slot="scope">
                  <el-button size="mini" type="primary" icon="el-icon-edit">编辑</el-button>
                  <el-button size="mini" type="danger" icon="el-icon-delete">删除</el-button>
                </template>
              </el-table-column>
            </el-table>
          </el-tab-pane>
           <!-- 添加静态属性的面板 将标签页改为only-->
            <el-tab-pane label="静态属性" name="only">
                <el-button size="mini" type="primary" :diabled="isButtonDisabled">添加属性</el-button>
                <!-- 静态属性表格 -->
                <el-table :data="onlyTableData" border stripe>
                    <!-- 展开行 -->
                    <el-table-column type="expand"></el-table-column>
                    <!-- 索引列 -->
                    <el-table-column type="index"></el-table-column>
                    <el-table-column label="属性名称" prop="attr_name"></el-table-column>
                    <el-table-column label="操作">
                        <template v-slot="scope">
                            <el-button size="mini" type="primary" icon="el-icon-edit">编辑</el-button>
                            <el-button size="mini" type="danger" icon="el-icon-delete">删除</el-button>
                        </template>
                    </el-table-column>
                </el-table>
            </el-tab-pane>
        </el-tabs>
        <el-dialog :title="'添加'+titleText" :visible.sync="addDialogVisible"
        width="50%" @close="addDialogClosed">
        <!-- 添加表单 -->
          <el-form :model="addForm" rules="addFormRules" ref="addFormRef"
          label-width="100px">
              <el-form-item :label="titleText" prop="attr_name">
                <el-input v-model="addForm.attr_name"></el-input>
              </el-form-item>
          </el-form>
          <span slot="footer" class="dialog-footer">
            <el-button @click="addDialogVisible=false">取消</el-button>
            <el-button @click="addParams">确定</el-button>
          </span>
        </el-dialog>
    </div>
</template>
<script>
export default {
  data(){
    return {
      cateList:[],
      //用户在级联下拉菜单中选中的分类id
      selectedCateKeys:[],
      //配置级联菜单中数据如何展示
      cateProps:{
        value:'cat_id',
        label:'cat_name',
        children:'children'
        },
      total:0,
      activeName:'many',
      manyTableData:[],
      onlyTableData:[],
      titleText:'',
      addDialogVisible:false,
      addForm:{
        attr_name:''
      },
      addFormRules:{
        attr_name:[{required:true,message:'请输入名称',trigger:'blur'}]
      }
    }
  },
  created(){
      this.getCateList()
  },
  methods:{
    async getCateList(){
      //获取所有的商品分类
      const {data:res} = await this.$http.get('categories')
      if(res.meta.status!==200){
        return this.$message.error('获取分类数据失败')
      }
      this.cateList = res.data
      this.total = res.data.total
    },
    async handleChange(){
      //当用户在级联菜单中选择内容改变时触发
      console.log(this.selectedCateKeys)
      const {data:res} = await this.$http.get(
          `categories/${this.cateId}/attributes`,
          {params:{sel:this.activeName}}
      )
      if(res.meta.status !==200) {
          return this.$message.error('获取参数列表数据失败')
      }
      if(this.activeName === "many"){
          //获取的是动态参数
          this.manyTableData = res.data
      }else if(this.activeName==="only"){
          //获取的静态属性
          this.onlyTableData = res.data
      }
    },
    handleTabClick(){
      this.handleChange()
    },
    addParams(){
      this.$refs.addFormRef.validate(async valid=>{
        if(!valid) return
      })
    }
  },
    computed:{
        //添加计算属性用来获取按钮禁用与否
        isButtonDisabled() {
            return this.selectedCateKeys.length!==3
        },
        //获取选中的三级分类
        cateId(){
            if(this.selectedCateKeys.length===3){
                return this.selectedCateKeys[this.selectedCateKeys.length-1]
            }
            return null
        }
    }
  }
</script>
<style lang="less" scoped>

</style>
