<template>
    <div class="login_container">
        <div class="center">
            <!-- <div class="one_img">
                <img src="../assets/logo.png">
            </div> -->
            <!-- 表单区域 -->
    <el-form ref="loginForm" :model="login_form" :rules="formRules" label-width="0px" class="one_form">
     <el-form-item prop="username">
      <el-input v-model="login_form.username" prefix-icon="el-icon-user"></el-input>
    </el-form-item>
     <el-form-item prop="password">
      <el-input v-model="login_form.password" prefix-icon="el-icon-warning"
      type="password"></el-input>
    </el-form-item>
    <el-form-item class="btns">
    <el-button type="primary" @click="login()">登录</el-button>
    <el-button type="primary" @click="resetForm()">重置</el-button>
    </el-form-item>
    </el-form>        
        </div>   
    </div>        
</template>
<script>
export default {
    data(){
        return {
            // 表单的数据绑定对象
            login_form:{
               username:'admin',
                password:'123456'
            },
            // 检查规则
            formRules:{
                // 验证用户是否合法
             username:[
                {required:true,message:'请输入用户',trigger:'blur'},
                {min:3,max:6,message:'长度在3到6个字符',trigger:'blur'}
              ],
            //   验证密码是否合法
               password:[
                {required:true,message:'请输入密码',trigger:'blur'},
                {min:3,max:6,message:'长度在6到14个字符',trigger:'blur'}
               ]
            }
        }
    },
    methods: {
      resetForm(){
        //   重置表单
         this.$refs.loginForm.resetFields();
      },
      login(){
        this.$refs.loginForm.validate(async val => {
            if (!val) return;
            // 解构赋值
           const { data:res } = await this.$http.post("login",this.login_form);
           console.log(this.login_form);
       if(res.meta.status !== 200)
        return this.$message.error("登录失败")
      this.$message.success("登录成功");
    //   1.将登录成功之后的token,保存到客户端的sessionStorage
    //   token只应在当前网站打开期间生效，所以将token保存在sessionStorage
    // 项目除了登录之外的别的API接口,必须在登录之后才能访问
      window.sessionStorage.setItem("token",res.data.token);
    // 通过编程式导航跳转到后台主页，
        this.$router.push("/home");
     }       
           
            
                   

        
       
        )  
      }  
    },
}
</script>

<style lang="less" scoped>
.login_container {
    // background-color:#2b4b6b;
    height: 100%;
    // background-color:#302866;
    background: #613f57;
}

.center{
    background-color: #fff;
    width: 450px;
    height: 300px;
    border-radius:3px;
    position: absolute;
    top:50%;
    left:50%;
    
    transform: translate(-50%,-50%);
.one_img{
     height: 130px;
     width: 130px;
     border:1px solid #eee;
     border-radius:50%;
     padding:10px;
     box-shadow: 0 0 10px #ddd;
     position:absolute;
     left:50%;
     transform: translate(-50%,-50%);
     background-color:#fff ;
     img{
         width: 100%;
         height: 100%;
         border-radius: 50%;
         background-color: #eee;
     }

    }
    .one_form{
        position: absolute;
        bottom: 0;
        width:100%;
        padding: 0 20px;
        box-sizing: border-box;

    }
    .btns{
        display: flex;
        justify-content: flex-end;
    }
}


</style>

