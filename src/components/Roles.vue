<template>
  <div>
    <h1>角色列表</h1>
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-table :data="rolesList" stripe>
      <el-table-column
        type="index">
      </el-table-column>
      <el-table-column
        label="角色名称"
        prop="roleName"
        >
      </el-table-column>
      <el-table-column
        label="角色描述"
        prop="roleDesc"
        >
      </el-table-column>
      <el-table-column
        label="权限等级"
        prop="level">
        <template v-slot="scope">
            <el-tag v-if = "scope.row.level === 0">一级权限</el-tag>
            <el-tag v-if = "scope.row.level === 1" type="success">二级权限</el-tag>
            <el-tag v-if = "scope.row.level === 2" type="warning">三级权限</el-tag>
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>

<script>
export default {
  data () {
    return {
      rolesList: []
    }
  },
  created () {
    this.getRolesList()
  },
  methods: {
    getRolesList () {
      const { data: res } = this.$http.get('roles')
      if (res.meta.status !== 200) return this.$message.error('权限获取失败')
      this.$message.success('权限获取成功')
      this.rolesList = res.data
    }
  }
}
</script>
<style lang="less" scoped>

</style>
