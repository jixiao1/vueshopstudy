<template>
<el-container class="home_container">
<el-header>
  <div>
    <img src="../assets/h1.png" alt="">
    <span>电商后台管理系统</span>
  </div>
  <el-button @click="logout" type="info">退出</el-button>
</el-header>
<!--  这里是主体区域 -->
  <el-container>
    <el-aside :width="iscollapse? '60px':'200px'">
      <div class="toggleMenu" @click="clickHandler">|||</div>
      <!-- 动态属性绑定 这里是侧边栏区域 -->
      <el-menu
      background-color="#333744"
      text-color="#fff"
      active-text-color="#409eff"
      unique-opened
      :collapse="iscollapse"
      :collapse-transition="false"
      :router= "true"
      :default-active="activePath"
      style="border:none;">
      <el-submenu :index="item.id + ''" v-for="item in menulist" :key="item.id">
        <template slot="title">
          <i :class="iconsObj[item.id]"></i>
          <span>{{item.authName}}</span>
        </template>
        <el-menu-item :index="'/'+subItem.path" v-for="subItem in item.children" :key="subItem.id" style="overflow:hidden;"
        @click="clickActivePath('/'+subItem.path)">
          <template slot="title"><i class="el-icon-menu"></i>{{subItem.authName}}</template>
        </el-menu-item>
      </el-submenu>
      </el-menu>
    </el-aside>
    <el-main>
      <!--  路由展示位 -->
      <router-view></router-view>
    </el-main>
  </el-container>
</el-container>
</template>
<script>
import axios from 'axios'
export default {
  data () {
    return {
      menulist: [],
      //  字体图标
      iconsObj: {
        '125': 'iconfont icon-user',
        '103': 'iconfont icon-tijikongjian',
        '101': 'iconfont icon-shangpin',
        '102': 'iconfont icon-danju',
        '145': 'iconfont icon-baobiao'
      },
      iscollapse: false,
      //  被激活的 地址
      activePath: ''
    }
  },
  methods: {
    logout() {
      window.sessionStorage.clear()
      this.$router.push('/login')
    },
    getMenuList () {
      // const { data: res } = await this.$http.get('menus')
      // if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      // console.log(res)
      axios.get('menus')
        .then(res => {
          // console.log(res.data.data)
          this.menulist = res.data.data
        })
        .catch(() => {
          alert('获取数据失败')
        })
    },
    clickHandler () {
      this.iscollapse = !this.iscollapse
    },
    clickActivePath (activePath) {
      window.sessionStorage.setItem('active', activePath)
      this.activePath = activePath
    }
  },
  created () {
    this.getMenuList()
    this.activePath = window.sessionStorage.getItem('active')
  }
}
</script>

<style lang="less" scoped>
.home_container {
  height: 100%;
}
.el-header{
  background-color: #373d41;
  display: flex;
  justify-content: space-between;
  align-items: center;
  >div {
    display: flex;
    align-items: center;
    span {
      color: #ffffff;
      font-size: 20px;
      margin-left: 20px;
    }
  }
}
.el-aside {
  background-color: #333744;
}
.el-main {
  background-color: #eaedf1;
}
.iconfont {
  margin-right: 10px;
}
.toggleMenu {
  background-color: #4a5064;
  color: #ffffff;
  text-align: center;
  font-size: 20px;
  //  文本中字和子之间的间隔
  letter-spacing: .2em;
  cursor: default;
}
.el-submenu__title {
  border: none;
  // background-color: red;
}
</style>
