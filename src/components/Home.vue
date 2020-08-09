<template>
  <el-container class="out_container" >
  <el-header>
      <div>
          <!-- <img src="../assets/logo.png" alt=""> -->
          <span>后台管理系统</span>
      </div>   
    <el-button type="info" @click="logout" class="logout">退出</el-button>
    </el-header>
    <!-- 页面主体区域 -->
  <el-container>
     <!-- 控制折叠 -->
    <el-aside :width="ifcollapse?'64px':'200px'">
        <!-- 侧边栏菜单区域 -->
        <div class="one-button" @click="if_collapse">|||</div>
        <el-menu 
     background-color="#613f57"
   
       
      text-color="#fff"
      active-text-color="#ffd04b " :unique-opened="true"
        
       :collapse="ifcollapse" :collapse-transition="false"
       :router='true' :default-active="activePath"> 
        <!-- collapse控制是否折叠 -->
      <!-- index控制所有的菜单是否一起动 -->
      <el-submenu :index="item.id+''"  v-for="item in menulist" :key=item.id >
        <template slot="title">
          <i :class="iconsObj[item.id]"></i>
          <span>{{item.authName}}</span>
        </template>
    <!-- 二级菜单 -->
    <el-menu-item :index="'/'+subitem.path" v-for="subitem in item.children" 
    :key=subitem.id @click="savaNavstatus('/'+subitem.path)">
      <template slot="title">
          <i class="el-icon-location"></i>
          <span>{{subitem.authName}}</span>
        </template></el-menu-item>
        
      </el-submenu>
      
    </el-menu>
    </el-aside>
    <el-main><router-view></router-view></el-main>
  </el-container>
</el-container>
       
    
</template>
<script>
export default {
  data() {
    // 左侧菜单数据
    return {
      menulist:[],
      // 每个一级菜单的图标
      iconsObj:{
        125:'el-icon-s-check',
        103:'el-icon-setting',
        101:'el-icon-present',
        102:'el-icon-date',
        145:'el-icon-document'
      },
      ifcollapse:false,
      activePath:'',
    }
    
  },
  created() {
    this.getMenuList();
    this.activePath = window.sessionStorage.getItem('activePath')
  },
    methods:{
        logout() {
            // 销毁token 
            window.sessionStorage.clear();
            this.$router.push("/login");
        },
        // 获取所有的菜单
        async getMenuList() {
          const {data:res} = await this.$http.get('menus');
          if(res.meta.status !== 200 ) return this.$message.error(res.meta.msg)
          this.menulist = res.data
          console.log(res)
        },
        // 控制属性来折叠菜单,
        if_collapse(){
          this.ifcollapse = !this.ifcollapse;
        },
        // 保存链接的激活状态
        savaNavstatus(activePath){
          // 将激活状态保存到sessionStorage
          window.sessionStorage.setItem('activePath',activePath);
          this.activePath = activePath
        }
    }    
}
</script>
<style lang="less" scoped>
.el-header {
// background-color:#333744;
// background-color:#302866;
// background-color:#3265e0;
//  background-color="#3265e0"
    // background-color:#302866;
    background: #613f57;
   

   


display:flex;
justify-content: space-between;
padding-left:0;
align-items:center;
color:#fff;
font-size:20px;
> div {
    display:flex;
    align-items:center;
    span {
        margin-left:15px;
    }
}

}
.el-aside {
// background-color:#333744;
// background-color:#302866;
// background-color:#3265e0;
    // background-color:#302866;
    background: #613f57;

//  background-color="#302866"
  


.el-menu {
  border-right: none;


}
}
.el-main{
    background: #EAEDF1;
    
}

.out_container{
    height: 100%;
}

.one-button {
  // background-color:#4a5064;
  // background-color:#613f57;
  background-color: #8f5c65;
  font-size:10px;
  line-height:24px;
  color:#fff;
  text-align:center;
  letter-spacing: 6px;
  //  background-color="#333744"
}
.logout{
  // background: #613f57;
  background-color: #8f5c65;

}
</style>