<template>
  <div>
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-table :data="rolesList" border>
      <el-table-column  type="expand">
        <template v-slot="scope">
          <el-row :class="['bdbottom',i1===0?'bdtop':'','vcenter']"
          v-for ="(item1,i1) in scope.row.children"
          :key ="item1.id">
              <!-- 渲染一级权限 -->
              <el-col :span="5">
                <el-tag closable @close="removeRightById(scope.row,item1.id)">
                  {{item1.authName}}
                </el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 渲染二，三级权限 -->
              <el-col :span="19">
                <!-- 通过for循环嵌套渲染二级权限 -->
                <el-row :class="[i2===0?'':'bdtop','vcenter']"
                v-for="(item2,i2) in item1.children"
                :key="item2.id">
                <el-col :span="6">
                  <el-tag type="success" closable @close="removeRightById(scope.row,item2.id)">
                    {{item2.authName}}
                  </el-tag>
                  <i class="el-icon-caret-right"></i>
                </el-col>
                <el-col :span="18">
                  <el-tag type="warning" v-for="
                  (item3) in item2.children"
                  :key="item3.id" closable @close="removeRightById(scope.row,item3.id)">
                      {{item3.authName}}
                  </el-tag>
                </el-col>
                </el-row>
              </el-col>
          </el-row>
        </template>
      </el-table-column>
      <el-table-column type="index" label="#"> </el-table-column>
      <el-table-column label="角色名称" prop="roleName" >  </el-table-column>
      <el-table-column label="角色描述" prop="roleDesc" > </el-table-column>
      <el-table-column label="操作" width="300px">
        <template v-slot="scope">
            <el-button size="mini" type="primary" icon="el-icon-edit" @click="showEditDialog(scope.row)">编辑</el-button>
            <el-button size="mini" type="danger" icon="el-icon-delete" @click="deleteRole(scope.row.id)">删除</el-button>
            <el-button size="mini" type="warning" icon="el-icon-setting" @click="showSetRightDialog(scope.row)">分配权限</el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 分配权限对话框 -->
    <el-dialog
      title="分配权限"
      :visible.sync="setRightDialogVisible"
      width="50%" @close="setRightDialogClose">
        <el-tree :data="rightsList" :props="treeProps"
        show-checkbox node-key="id" default-expand-all
        :default-checked-keys="defKeys" ref="treeRef">
        </el-tree>
        <span slot="footer" class="dialog-footer">
          <el-button @click="setRightDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="allotRights">确 定</el-button>
        </span>
    </el-dialog>
<!-- 编辑角色对话框 -->
    <el-dialog
    title="编辑角色" :visible.sync="setEditDialogVisible"
    width="50%" @close="setEditDialogClose">
      <el-form :model="editForm" ref="editFormRef" :rules="editFormRules">
        <!-- 角色名称 -->
        <el-form-item prop="roleName">
          <el-input v-model="editForm.roleName"></el-input>
        </el-form-item>
        <!-- 角色描述 -->
        <el-form-item prop="roleDesc">
          <el-input v-model="editForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setEditDialogVisible=false">取 消</el-button>
        <el-button type="primary" @click="editRole">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      rolesList: [],
      setRightDialogVisible: false,
      rightsList: [],
      // 树形控件的属性绑定对象
      treeProps: {
        label: 'authName', // 通过label设置树形节点文本展示authName
        children: 'children'// 设置通过children属性展示子节点信息
      },
      defKeys: [], // 设置树形控件中默认选中的内容
      roleId: '', // 保存正在操作的角色id
      setEditDialogVisible:false,
      editForm:{
        role_name:'',
        role_desc:''
      },
      editFormRules:{
        roleName:[
          {required:true,message:'请输入角色名称',trigger:'blur'}
        ]
      }
    }
  },
  created () {
    this.getRolesList()
  },
  methods: {
    async getRolesList () {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) return this.$message.error('权限获取失败')
      this.$message.success('权限获取成功')
      this.rolesList = res.data
    },
    async removeRightById (role, rightId) {
      const confirmResult = await this.$confirm('确认删除该权限吗?', '删除提示', {
        confirmButtonText: '确认',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (confirmResult !== 'confirm') {
        return this.$message.info('已经取消删除')
      }
      const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
      if (res.meta.status !== 200) return this.$message.error('删除权限失败')
      this.$message.success('删除权限成功')
      role.children = res.data
    },
    async showSetRightDialog (role) {
      this.roleId = role.id // 将role.id保存起来以供保存权限时使用
      // 1.获取所有权限的数据
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) return this.$message.error('获取权限树失败')
      this.rightsList = res.data
      // 2.调用getLeafKeys进行递归，将三级权限添加到数组中
      this.getLeafKeys(role, this.defKeys)
      this.setRightDialogVisible = true
    },
    getLeafKeys (node, arr) {
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item => this.getLeafKeys(item, arr))
    },
    setRightDialogClose () {
      this.defKeys = []
    },
    async allotRights () {
      const keys = [...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()]
      const idStr = keys.join(',')
      const { data: res } = await this.$http.post(
        `roles/${this.roleId}/rights`,
        { rids: idStr }
      )
      if (res.meta.status !== 200) return this.$message.error('分配权限失败')
      this.$message.success('分配权限成功')
      this.getRolesList()
      this.setRightDialogVisible = false
    },
    async showEditDialog(role){
      this.roleId = role.id
      const {data: res}  = await this.$http.get('roles/'+this.roleId)
      if(res.meta.status!==200) return this.$message.error('角色信息获取失败')
      this.editForm = res.data
      this.setEditDialogVisible=true
    },
    setEditDialogClose(){
      this.$refs.editFormRef.resetFields()
    },
    editRole(){
      this.$refs.editFormRef.validate(async valid=>{
        if(!valid) return
        const {data: res} = await this.$http.put(`roles/${this.roleId}`,this.editForm)
        if(res.meta.status!==200) this.$message.error('修改角色失败')
        this.$message.success('修改角色成功')
        this.$refs.editFormRef.resetFields()
        this.getRolesList()
        this.setEditDialogVisible= false
      })
    },
    async deleteRole(roleId){
      const confirmResult = await this.$confirm('确认删除该角色吗?','删除提示',{
        confirmButtonText:'确认',
        cancelButtonText:'取消',
        type:'warning'
      }).catch(err=>err)
      if(confirmResult!='confirm') return this.$message.info('已取消删除操作')
      const {data: res} = await  this.$http.delete(`roles/${roleId}`)
      if(res.meta.status!==200) return this.$message.error('删除失败')
      this.$message.success('删除成功')
      this.getRolesList()
      
    }
  }
}
</script>
<style lang="less" scoped>
  .el-tag{margin:7px;}
  .bdtop{border-top:1px solid #eee}
  .bdbottom{border-bottom:1px solid #eee}
</style>
