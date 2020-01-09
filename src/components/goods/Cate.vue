<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{path: '/home'}">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
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
      <tree-table :data="cateList" :columns="columns" :selection-type="false"
      :expand-type="false" show-index index-text="#" border :show-row-hover="false">
      <!-- 是否有效区域 -->
        <template v-slot:isok="scope">
          <i class="el-icon-success" v-if="scope.row.cat_deleted===false"
          style="color:lightgreen;"></i>
          <i class="el-icon-error" v-else style="color:red;"></i>
        </template>
      <!-- 排序 -->
      <template v-slot:order="scope">
        <el-tag size="mini" v-if="scope.row.cat_level===0">一级</el-tag>
        <el-tag size="mini" type="success" v-else-if="scope.row.cat_level===1">二级</el-tag>
        <el-tag size="mini" type="warning" v-else>三级</el-tag>
      </template>
      <!-- 操作 -->
      <template v-slot:opt="scope">
          <el-button size="mini" type="primary" icon="el-icon-edit">编辑</el-button>
          <el-button size="mini" type="danger" icon="el-icon-delete">删除</el-button>
      </template>
      </tree-table>
      <!-- 分页 -->
      <el-pagination @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="queryInfo.pagenum"
      :page-sizes="[3,5,10,15]"
      :page-size="queryInfo.pagesize"
      layout="total,sizes,prev,pager,next,jumper"
      :total="total">
      </el-pagination>
    </el-card>
    <el-dialog title="添加分类" :visible.sync="addCateDialogVisible" width="50%"
    @close="addCateDialogClosed">
    <!-- 添加分类表单 -->
    <el-form :model="addCateForm" :rules="addCateFormRules" ref="addCateFormRef"
    label-width="100px">
      <el-form-item label="分类名称" prop="cat_name">
        <el-input v-model="addCateForm.cat_name"></el-input>
      </el-form-item>
      <el-form-item label="父级分类" prop="cat_pid">
        <el-cascader expandTrigger='hover' v-model="selectedKeys" :options="parentCateList"
        :props="cascaderProps" @change="parentCateChange" clearable
        change-on-select>

        </el-cascader>
      </el-form-item>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addCateDialogVisible=false">取消</el-button>
        <el-button type="primary" @click="addCate">确定</el-button>
      </span>
    </el-form>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data () {
    return {
      cateList: [],
      queryInfo: {
        type: 3,
        pagenum: 1,
        pageSize: 5
      },
      total: 0,
      columns: [
        { label: '分类名称', prop: 'cat_name' },
        { label: '是否有效', type: 'template', template: 'isok' },
        { label: '排序', type: 'template', template: 'order' },
        { label: '操作', type: 'template', template: 'opt' }
      ],
      addCateDialogVisible: false,
      addCateForm: {
        cat_name: '',
        cat_pid: 0,
        cat_level: 0
      },
      addCateFormRules: {
        cat_name: [{ required: true,
          message: '请输入分类名称',
          trigger: 'blur' }]
      },
      parentCateList: [],
      cascaderProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children',
        expandTrigger: 'hover'
      },
      selectedKeys: []
    }
  },
  created () {
    this.getCateList()
  },
  methods: {
    async getCateList () {
      const { data: res } = await this.$http.get('categories', {
        params: this.queryInfo
      })
      if (res.meta.status !== 200) return this.$message.error('获取商品列表数据失败')
      this.cateList = res.data
      this.total = res.data.length
    },
    handleSizeChange (newSize) {
      this.queryInfo.pagesize = newSize
      this.getCateList()
    },
    handleCurrentChange (newPage) {
      this.queryInfo.pagenum = newPage
      this.getCateList()
    },
    showAddCateDialog () {
      this.getParentCateList()
      this.addCateDialogVisible = true
    },
    async getParentCateList () {
      const { data: res } = await this.$http.get('categories', {
        params: { type: 2 }
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类列表数据失败')
      }
      console.log(res)
      this.parentCateList = res.data
    },
    parentCateChange () {
      console.log(this.selectedKeys)
      if (this.selectedKeys.length > 0) {
        this.addCateForm.cat_pid = this.selectedKeys[this.selectedKeys.length - 1]
        this.addCateForm.cat_level = this.selectedKeys.length
      } else {
        this.addCateForm.cat_pid = 0
        this.addCateForm.cat_level = 0
      }
    },
    addCateDialogClosed () {
    // 当关闭添加分类对话框时，重置表单
      this.$refs.addCateFormRef.resetFields()
      this.selectedKeys = []
      this.addCateForm.cat_pid = 0
      this.addCateForm.cat_level = 0
    },
    addCate () {
    // 点击确定，完成添加分类
      console.log(this.addCateForm)
      this.$refs.addCateFormRef.validate(async valid => {
        if (!valid) return
        // 发送请求完成添加分类
        const { data: res } = await this.$http.post(
          'categories',
          this.addCateForm
        )

        if (res.meta.status !== 201) {
          return this.$message.error('添加分类失败')
        }

        this.$message.success('添加分类成功')
        this.getCateList()
        this.addCateDialogVisible = false
      })
    }
  }
}
</script>
<style lang="less" scoped>

</style>
