<template>
  <div class="login_container">
    <div class="login_box">
      <!-- 头像区域 -->
      <div class="avatar_box">
        <img src="../assets/logo.png" alt="" />
      </div>
      <!-- 表单区域 -->
      <el-form ref="loginFormRef" :rules="loginFormRules" :model="loginForm" label-width="0" class="login_form">
        <!-- 用户名 -->
        <el-form-item prop="username">
          <el-input v-model="loginForm.username" prefix-icon="iconfont icon-user"></el-input>
        </el-form-item>
        <!-- 密码 -->
        <el-form-item prop="password">
          <el-input v-model="loginForm.password" type="password" prefix-icon="iconfont icon-3702mima"></el-input>
        </el-form-item>
        <!-- 按钮区域 -->
        <el-form-item class="btns">
            <el-button @click="login" type="primary">登录</el-button>
            <el-button @click="resetLoginForm" type="info">重置</el-button>
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>

<script>
export default {
    data() {
        return {
            // 这是登录表单的数据绑定对象
            loginForm: {
                username: 'admin',
                password: '123456'
            },
            // 这是表单验证规则对象
            loginFormRules: {
                // 验证用户名是否合法
                username: [
                    { required: true, message: '请输入登录名称', trigger: 'blur' },
                    { min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' }
                ],
                // 验证密码是否合法
                password: [
                    { required: true, message: '请输入登录密码', trigger: 'blur' },
                    { min: 6, max: 15, message: '长度在 6 到 15 个字符', trigger: 'blur' }
                ]
            }
        }
    },
    methods: {
        resetLoginForm() {
            // 点击重置按钮，重置登录表单
            this.$refs.loginFormRef.resetFields()
        },
        login() {
            this.$refs.loginFormRef.validate( async valid => {
                if(!valid) return;
                // 如果某个方法的返回值是Promise，我们可以用async await简化这次操作
                // 紧挨着await的那个方法添加async
                // 从 返回值 身上通过{}结构赋值出data属性 把他重命名为res
                const {data: res} = await this.$http.post('login', this.loginForm);
                if(res.meta.status !== 200) return console.log('登录失败');
                console.log('登录成功');
               
                

            });
        }
    }
};
</script>

<style lang="less" scoped>
.login_container {
  background-color: #2b4b6b;
  height: 100%;
}
.login_box {
  width: 450px;
  height: 300px;
  background-color: #fff;
  border-radius: 3px;
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);

  .avatar_box {
    height: 130px;
    width: 130px;
    border: 1px solid #eee;
    border-radius: 50%;
    // 图片与边框之间有间距
    padding: 10px;
    box-shadow: 0 0 10px #ddd;
    position: absolute;
    left: 50%;
    transform: translate(-50%, -50%);
    background-color: #fff;
    img {
      width: 100%;
      height: 100%;
      // 图片加圆角效果
      border-radius: 50%;
      background-color: #eee;
    }
  }
  
  .login_form {
    position: absolute;
    bottom: 0;
    width: 100%;
    padding: 0 20px;
    // 默认form表单是传统盒子模型 content-box 设置成CSS3盒子模型
    box-sizing: border-box;
  }
  
  .btns {
    display: flex;
    // 尾部对齐
    justify-content: flex-end;
  }

  
}
</style>

