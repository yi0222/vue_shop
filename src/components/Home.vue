<template>
  
  <el-container class="home_container">
    <!-- 头部区域 -->
    <el-header>
      <div>
        <img src="../assets/logo1.png" alt="">
        <span>电商后台管理系统</span>
      </div>
      <el-button type="info" @click="logout">退出</el-button>
    </el-header>
    <!-- 主体区域 -->
    <el-container>
      <!-- 侧边栏 -->
      <el-aside :width="isCollapse ? '64px':'200px'">
        <div class="toggle-button" @click="toggleCollapse">|||</div>
        <el-menu background-color="#333744" text-color="#fff" active-text-color="#409EFF" 
        :unique-opened="true" :collapse="isCollapse" :collapse-transition="false" :router="true" :default-active="activePath">
          <!-- 一级菜单 -->
          <el-submenu :index="item.id+''" v-for="item in menuList" :key="item.id">
            <!-- 一级菜单的模板区域 -->
            <template slot="title">
              <!-- 图标 -->
              <i :class="iconList[item.id]"></i>
              <!-- 文本 -->
              <span>{{item.authName}}</span>
            </template>

            <!-- 二级菜单 -->
            <el-menu-item :index="'/'+subItem.path" v-for="subItem in item.children" :key="subItem.id" @click="activeItem('/'+subItem.path)">
              <template slot="title">
                <!-- 图标 -->
                <i class="el-icon-menu"></i>
                <!-- 文本 -->
                <span>{{subItem.authName}}</span>
              </template>
            </el-menu-item>
          </el-submenu>
        </el-menu>
      </el-aside>
      <!-- 右侧主体 -->
    <el-main>
      <router-view></router-view>
    </el-main>
    </el-container>
  </el-container>
</template>

<script>
export default {
  data(){
    return{
      menuList:[],
      iconList:{
        '125': 'iconfont icon-user',
        '103': 'iconfont icon-tijikongjian',
        '101': 'iconfont icon-shangpin',
        '102': 'iconfont icon-danju',
        '145': 'iconfont icon-baobiao'
      },
      isCollapse:false,
      activePath:''

    }
  },
  created(){
    this.getMenuList();
    this.activePath=window.sessionStorage.getItem('activePath')
  },
  methods:{
    logout(){
      window.sessionStorage.clear();
      this.$router.push('/login')
    },
    async getMenuList(){
      const {data:res}= await this.$http.get('menus');
      console.log(res);
      if(res.meta.status !==200) return this.$message.error(res.meta.msg);
      this.menuList = res.data     
    },
    toggleCollapse(){
      this.isCollapse=!this.isCollapse
    },
    activeItem(activePath){
      window.sessionStorage.setItem('activePath',activePath);
      this.activePath = activePath
    }
  }
}
</script>

<style lang="less" scoped>
  .home_container{
    height: 100%;
  }
  .el-header{
    background-color: #373d41;
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding-left: 0;
    color: #fff;
    font-size: 20px;
    div {
      display: flex;
      align-items: center;
      span{
        margin-left: 15px;
      }
    }
  }
  .el-aside{
    background-color: #333744;
    .el-menu{
      border: 0;
    };
    .toggle-button{
      text-align: center;
      background-color: #4A5064;
      line-height: 24px;
      color: #fff;
      font-size: 10px;
      letter-spacing: 0.2em;
      cursor: pointer;
    }
  }
 .el-maim{
  background-color: #eaedf1;
  }
  .iconfont {
    margin-right: 10px;
  }
</style>