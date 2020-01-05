<template>
    <el-container class="home-container">
      <!-- 头部区域 -->
    <el-header>
      <div>
        <img src="../assets/heima.png" alt="">
        <span>电商后台管理系统</span>
      </div>
      <el-button type="info" @click="logout">退出</el-button>
    </el-header>
    <!-- 页面主体区域 -->
    <el-container>
      <!-- 侧边栏 -->
      <el-aside :width="isCollapse?'64px':'200px'">
         <div class="toggle-button" @click="toggleCollapse">|||</div>
         <el-menu
      background-color="#323744"
      text-color="#fff"
      active-text-color="#359EFF"
      unique-opened :collapse="isCollapse"
      :collapse-transition="false"
      router>
      <!-- 一级菜单 -->
      <el-submenu :index="item.id+''" v-for="item in menuList" :key="item.id">
        <!-- 一级菜单模板 -->
        <template slot="title">
          <!-- 图标 -->
          <i :class="iconList[item.id]"></i>
          <!-- 文字 -->
          <span>{{item.authName}}</span>
        </template>
        <el-menu-item :index="'/'+subItem.path" v-for="subItem in item.children" :key="subItem.id">
          <template slot="title">
          <!-- 图标 -->
          <i class="el-icon-menu"></i>
          <!-- 文字 -->
          <span>{{subItem.authName}}</span>
          </template>
        </el-menu-item>
      </el-submenu>
    </el-menu>
      </el-aside>
      <!-- 内容区域 -->
      <el-main>
        <router-view/>
      </el-main>
    </el-container>
  </el-container>
</template>

<script>
export default {
  data () {
    return {
      menuList: [],
      iconList: {
        '125': 'iconfont icon-user',
        '103': 'iconfont icon-tijikongjian',
        '101': 'iconfont icon-shangpin',
        '102': 'iconfont icon-danju',
        '145': 'iconfont icon-baobiao'
      },
      isCollapse: false
    }
  },
  created () { this.getMenuList() },
  methods: {
    logout () {
      window.sessionStorage.clear()
      this.$router.push('/login')
    },
    // 获取菜单数据
    async getMenuList () {
      const { data: res } = await this.$http.get('menus')
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.menuList = res.data
    },
    // 折叠菜单栏
    toggleCollapse () {
      this.isCollapse = !this.isCollapse
    }
  }
}
</script>

<style lang="less" scoped>
.home-container {
  height: 100%;
}
.el-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  color:#fff;
  font-size: 20px;
  background-color: #373C41;
  > div {
    display: flex;
    align-items: center;
    span {
      margin-left: 10px;
    }

  }
}
.el-aside {
 background-color:#323744;
 .toggle-button{
     font-size: 15px;
     color:#fff;
     line-height: 30px;
     height: 30px;
     text-align: center;
     letter-spacing: .2em;
     background-color: #4A5065;
     cursor: pointer;
   }
 .el-menu {
   border-right:none;
 }
}
.el-main {
  background-color: #EAEDF2;
}
.iconfont {
  padding-right: 10px;
}
</style>
