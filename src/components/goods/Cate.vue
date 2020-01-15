<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{path:'/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>分类管理</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card>
      <!-- 添加分类按钮区域 -->
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAddCateDialog">添加分类</el-button>
        </el-col>
      </el-row>
      <!-- 分类表格 -->
      <tree-table class="treeTable" :data="cateList" :columns="columns" :selection-type="false" :expand-type="false"
        show-index index-text="#" border :show-row-hover="false">
        <template v-slot:isok="scope">
          <i class="el-icon-success" v-if="scope.row.cat_deleted===false" style="color:lightgreen"></i>
        </template>
        <!-- 排序 -->
        <template v-slot:order="scope">
          <el-tag size="mini" v-if="scope.row.cat_level===0">一级</el-tag>
          <el-tag size="mini" type="success" v-else-if="scope.row.cat_level===1">二级</el-tag>
          <el-tag size="mini" type="warning" v-else>三级</el-tag>
        </template>
        <!-- 操作 -->
        <template v-slot:opt="scope">
          <el-button size="mini" type="primary" icon="el-icon-edit" @click="showEditCateDialog(scope.row.cat_id)">编辑
          </el-button>
          <el-button size="mini" type="danger" icon="el-icon-delete" @click="deleteCate(scope.row.cat_id)">删除</el-button>
        </template>
      </tree-table>
      <!-- 分页 -->
      <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum" :page-sizes="[3,5,10,15]" :page-size="queryInfo.pagesize"
        layout="total,sizes,prev,pager,next,jumper" :total="total"></el-pagination>
    </el-card>
    <!-- 添加分类对话框 -->
    <el-dialog title="添加分类" :visible.sync="addCateDialogVisible" width="50%" @close="addCateDialogClosed">
      <!-- 添加分类表单 -->
      <el-form :model="addCateForm" :rules="addCateFormRules" ref="addCateFormRef" label-width="100px">
        <el-form-item label="分类名称" prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级名称">
          <el-cascader class="el-cascader" :options="parentCateList" :props="cascaderProps" v-model="selectedKeys"
            @change="parentCateChange" clearable>
          </el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addCateDialogVisible=false">取消</el-button>
        <el-button type="primary" @click="addCate">确定</el-button>
      </span>
    </el-dialog>
    <!-- 修改分类对话框 -->
    <el-dialog title="修改分类" :visible.sync="editCateDialogVisible" width="50%" @close="editCateDialogClosed">
      <!-- 修改分类表单 -->
      <el-form :model="editCateForm" :rules="editCateFormRules" ref="editCateFormRef" label-width="100px">
        <el-form-item label="分类名称" prop="cat_name">
          <el-input v-model="editCateForm.cat_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editCateDialogVisible=false">取消</el-button>
        <el-button type="primary" @click="editCate">确定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
  export default {
    data() {
      return {
        cateList: [],
        queryInfo: {
          type: 3,
          pagenum: 1,
          pagesize: 5
        },
        total: 0,
        columns: [{
            label: '分类名称',
            prop: 'cat_name'
          },
          {
            label: '是否有效',
            prop: '',
            type: 'template',
            template: 'isok'
          },
          {
            label: '排序',
            prop: '',
            type: 'template',
            template: 'order'
          },
          {
            label: '操作',
            prop: '',
            type: 'template',
            template: 'opt'
          }
        ],
        addCateDialogVisible: false,
        addCateForm: {
          cat_name: '',
          cat_pid: 0,
          cat_level: 0
        },
        addCateFormRules: {
          cat_name: [{
            required: true,
            message: '请输入分类名称',
            trigger: 'blur'
          }]
        },
        parentCateList: [],
        cascaderProps: {
          value: 'cat_id',
          label: 'cat_name',
          children: 'children',
          expandTrigger: 'hover',
          checkStrictly: 'true'
        },
        selectedKeys: [],
        editCateDialogVisible:false,
        editCateForm:{
          cat_id:0,
          cat_name: '',
        },
        editCateFormRules:{
          cat_name:[{
            required:true,
            message:'请输入分类名称',
            trigger:'blur'
          }]
        }
      } 
    },
    created() {
      this.getCateList()
    },
    methods: {
      async getCateList() {
        const {
          data: res
        } = await this.$http.get('categories', {
          params: this.queryInfo
        })
        if (res.meta.status !== 200) return this.$message.error('获取分类列表失败')
        this.cateList = res.data.result
        this.total = res.data.total
      },
      handleSizeChange(newSize) {
        this.queryInfo.pagesize = newSize
        this.getCateList()
      },
      handleCurrentChange(newPage) {
        this.queryInfo.pagenum = newPage
        this.getCateList()
      },
      showAddCateDialog() {
        this.getParentCateList()
        this.addCateDialogVisible = true
      },
      async getParentCateList() {
        const {
          data: res
        } = await this.$http.get('categories', {
          params: {
            type: 2
          }
        })
        if (res.meta.status !== 200) return this.$message.error('获取分类数据失败')
        console.log(res.data)
        this.parentCateList = res.data
      },
      addCateDialogClosed() {
        this.$refs.addCateFormRef.resetFields()
        this.selectedKeys = []
        this.addCateForm.cat_pid = 0
        this.addCateForm.cat_level = 0
        this.addCateDialogVisible = false
      },
      parentCateChange() {
        if (this.selectedKeys.length > 0) {
          this.addCateForm.cat_pid = this.selectedKeys[this.selectedKeys.length - 1]
          this.addCateForm.cat_level = this.selectedKeys.length
          return
        } else {
          this.addCateForm.cat_pid = 0
          this.addCateForm.cat_level = 0
        }
      },
      addCate() {
        this.$refs.addCateFormRef.validate(async valid => {
          if (!valid) return
          const {
            data: res
          } = await this.$http.post('categories', this.addCateForm)
          if (res.meta.status !== 201) return this.$message.error('新增分类失败')
          this.$message.success('新增分类成功')
          this.getCateList()
          this.addCateDialogVisible = false
        })
      },
      //显示修改对话框
      async showEditCateDialog(id){
          const {data:res} = await this.$http.get('categories/'+id)
          if(res.meta.status!==200) return this.$message.error('获取分类失败')
          this.editCateForm = res.data
          this.editCateDialogVisible=true
      },
      editCate(){
         this.$refs.editCateFormRef.validate( async valid=>{
          if(!valid) return this.$message.error('请输入分类名称')
          const {data:res} = await this.$http.put('categories/'+this.editCateForm.cat_id,this.editCateForm)
          if(res.meta.status!==200) return this.$message.error('修改分类失败')
          this.editCateDialogVisible=false
          this.getCateList()
        })
      },
      editCateDialogClosed(){
        this.$refs.editCateFormRef.resetFields()
        this.editCateDialogVisible=false
      },
      async deleteCate(id){
        const confirmResult = await this.$confirm("确认永久删除该分类吗",{
          confirmButtonText:'确认删除',cancelButtonText:'取消',type:'warning'
        }).catch(error=>error)
        if(confirmResult!=='confirm') return this.$message.info('已经取消删除')
        const {data:res} = await this.$http.delete('categories/'+id)
        if(res.meta.status!==200) return this.$message.error('删除失败')
        this.$message.success('删除成功')
        this.getCateList()
      }
    }
  }

</script>
<style lang="less" scoped>
<<<<<<< HEAD
.treeTable {
  margin-top: 15px;
}
=======
  .treeTable {
    margin-top: 15px;
  }

  .el-cascader-panel {
    width: 100%;
  }
>>>>>>> b60ce11009e5aa04461685c3dc2d88ac80ef7c71

.el-cascader-panel {
  width: 100%;
}
.el-popper {
  top:0 !important;
}
.el-cascader-node {
  height: 25px;
}
</style>
