<template>
  <div>
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-table row-key="id" :data="rolesList" stripe>
      <!-- 添加展开列 -->
      <el-table-column
        type="expand">
      </el-table-column>
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
        label="操作"
        width="300px">
        <template v-slot="scope">
            <el-button size="mini" type="primary" icon="el-icon-edit">编辑</el-button>
            <el-button size="mini" type="danger" icon="el-icon-delete">删除</el-button>
            <el-button size="mini" type="warning" icon="el-icon-setting">分配权限</el-button>
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
    async getRolesList () {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) return this.$message.error('角色获取失败')
      this.$message.success('角色获取成功')
      this.rolesList = res.data
    }
  }
}
</script>
<style lang="less" scoped>

</style>
