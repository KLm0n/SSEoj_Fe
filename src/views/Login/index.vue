<!-- eslint-disable vue/multi-word-component-names -->
<template>
  <div class="container">
    <header>
      <h2 class="logo">SSEOJ</h2>
    </header>
    <background style="position: fixed; top: 100px; left: 30px; width: 500px;"/>
    <div class="pane">
      <div class="wrapper" :class="{ active }">
        <!-- Login Form -->
        <div class="form_box login">
          <h2>Login</h2>
          <form @submit.prevent="doLogin">
            <div class="input_box">
              <span class="icon"><ion-icon name="mail"></ion-icon></span>
              <input type="text" v-model="email" required />
              <label>Email</label>
            </div>
            <div class="input_box">
              <span class="icon"><ion-icon name="lock-closed"></ion-icon></span>
              <input type="password" v-model="password" required />
              <label>Password</label>
            </div>
            <div class="remember_forgot">
              <a href="#" class="forgot-password" @click="dialogFormVisible=true">Forgot Password?</a>
            </div>
            <button type="submit" class="btn">Login</button>
            <div class="login_register">
              <p>Don't have an account? <label class="register_link" @click="showRegister">Register</label></p>
            </div>
          </form>
        </div>

        <!-- Registration Form -->
        <div class="form_box register">
          <h2>Registration</h2>
          <form>
            <div class="input_box">
              <span class="icon"><ion-icon name="person"></ion-icon></span>
              <input type="text" v-model="username_register" required>
              <label>Username</label>
            </div>
            <div class="input_box">
              <span class="icon"><ion-icon name="mail"></ion-icon></span>
              <input type="text" v-model="email_register" required>
              <label>Email</label>
            </div>
            <div class="input_box_verfication">
              <span class="icon"><ion-icon name="pencil"></ion-icon></span>
              <input type="text" v-model="verificationCode_register" required>
              <label>Verification Code</label>
            </div>
            <div>
              <button type="submit" class="btn_verification" @click="sendVerification(email_register, 0)">Send</button>
            </div>
            <div class="input_box">
              <span class="icon"><ion-icon name="lock-closed"></ion-icon></span>
              <input type="password" v-model="password_register" required>
              <label>Password</label>
            </div>
            <div class="input_box">
              <span class="icon"><ion-icon name="alert-circle"></ion-icon></span>
              <input type="password" v-model="repassword" required>
              <label>Repassword</label>
            </div>
            <button type="sumit" class="btn" @click="doRegister">Register</button>
            <div class="login_register">
              <p>Already have an account? <label class="login_link" @click="showLogin">Login</label></p>
            </div>
          </form>
        </div>
      </div>
    </div>

    <el-dialog v-model="dialogFormVisible" title="Fogotten Your Passwords?" width="500" top="300px" class="forgetContainer">
      <el-form :model="form">
        <el-form-item label="邮箱" :label-width="formLabelWidth">
          <el-input v-model="email_forgot" autocomplete="off" />
        </el-form-item>
        <el-form-item v-model="verificationCode_forgot" label="邮箱验证码" :label-width="formLabelWidth"  class="emailVerification">
          <el-input v-model="verificationCode_forgot" autocomplete="off"/>
          <el-button
            :disabled="isDisabled"
            type="success"
            plain
            class="btnSend"
            @click="sendVerification(email_forgot,1)">
            {{ isDisabled ? countdown + '秒' : '发送' }}
          </el-button>
        </el-form-item>
        <el-form-item label="新密码" :label-width="formLabelWidth">
          <el-input v-model="forgetPassword" autocomplete="off" />
        </el-form-item>
        <el-form-item label="再输入一遍" :label-width="formLabelWidth">
          <el-input v-model="forgetPassword_again" autocomplete="off" />
        </el-form-item>
      </el-form>
      <template #footer>
        <div class="dialog-footer">
          <el-button @click="dialogFormVisible = false">Cancel</el-button>
          <el-button type="primary" @click="handleForgotPassword">
            Confirm
          </el-button>
        </div>
      </template>
    </el-dialog>
  </div>
  
</template>

<script setup>
import background from '../Layout/components/background.vue';
import { ref, onMounted } from 'vue';
import { ElMessage } from 'element-plus';
import { useUserStore } from '@/stores/userStore';
import { getIdentityAPI, registerAPI } from '@/apis/user';
import 'element-plus/theme-chalk/el-message.css';
import router from '@/router';
import forge from 'node-forge';
import { END_ALIGNMENT } from 'element-plus/es/components/virtual-list/src/defaults';

// Data properties
const email = ref('');
const password = ref('');
const username_register = ref('');
const email_register = ref('');
const password_register = ref('');
const verificationCode_register = ref('');
const repassword = ref('');
const active = ref(false);

const userStore = useUserStore()

const dialogFormVisible=ref(false);
const formLabelWidth = '100px'
const email_forgot=ref('');
const verificationCode_forgot=ref('');
const forgetPassword=ref('')
const forgetPassword_again=ref('')
// 定义响应式变量
const isDisabled = ref(false);  // 按钮是否禁用
const countdown = ref(15);      // 倒计时秒数
let timer = null;               // 用于存储定时器

const sendVerification =async(email,type)=>{
  if (isDisabled.value) return;  // 如果按钮已经禁用，点击无效
  if(email.trim()){
    try {
      await getIdentityAPI({email:email,type:type});
      ElMessage.success('验证码发送成功');

      // 开始倒计时
      isDisabled.value = true;
      timer = setInterval(() => {
        if (countdown.value > 0) {
          countdown.value--;
        } else {
          clearInterval(timer); // 倒计时结束，清除定时器
          isDisabled.value = false; // 重新启用按钮
          countdown.value = 15; // 重置倒计时
        }
      }, 1000);
    } catch (error) {
      ElMessage.error('验证码发送失败');
    }
  }
  else{
    ElMessage.error('邮箱不能为空!!!');
  }
}

const handleForgotPassword = async () => {

  if(!email_forgot.value.trim()||!verificationCode_forgot.value.trim()||!forgetPassword.value.trim()||!forgetPassword_again.value.trim()){
    ElMessage.error('请输入完整信息!!')
  }
  else{
    if(forgetPassword.value===forgetPassword_again.value){
      const publicKeyPem = `-----BEGIN PUBLIC KEY-----
    MFwwDQYJKoZIhvcNAQEBBQADSwAwSAJBAKHP3hnFtL1g3bpgDMFAma1MofY9UmInthR8vK5Q9/dYcfdqvzLFRdRCPdeefqGO+BIFfLeCKJi4odn61XJEvp8CAwEAAQ==
    -----END PUBLIC KEY-----`
      const publicKey = forge.pki.publicKeyFromPem(publicKeyPem);
      const encrypted = forge.util.encode64(publicKey.encrypt(forgetPassword.value));
      console.log(encrypted)
      const res = await userStore.passwordForget({email: email_forgot.value, password_new: encrypted, verification_code:verificationCode_forgot.value});
      console.log(res)
      if(res){
        ElMessage.success('密码修改成功！');
        dialogFormVisible.value = false;
      }
      else{
        ElMessage.error('密码修改失败！');
      }
    }
    else{
      ElMessage.error('两次输入的密码不一致！')

    }
  }

};

// Methods
const doLogin = async () => {
  const publicKeyPem = `-----BEGIN PUBLIC KEY-----
MFwwDQYJKoZIhvcNAQEBBQADSwAwSAJBAKHP3hnFtL1g3bpgDMFAma1MofY9UmInthR8vK5Q9/dYcfdqvzLFRdRCPdeefqGO+BIFfLeCKJi4odn61XJEvp8CAwEAAQ==
-----END PUBLIC KEY-----`
  const publicKey = forge.pki.publicKeyFromPem(publicKeyPem);
  const encrypted = forge.util.encode64(publicKey.encrypt(password.value));
  console.log(encrypted)
  await userStore.getUserInfo(email.value, password.value)
  ElMessage({ type: "success", message: "登陆成功" })
  const redirectPath = localStorage.getItem('redirectPath') || '/'
  router.replace(redirectPath)
  localStorage.removeItem('redirectPath')
}

const doRegister = async() =>{
  if(!username_register.value.trim() || !email_register.value.trim() || !password_register.value.trim() || !repassword.value.trim() || !verificationCode_register.value.trim()){
    ElMessage.error('注册信息不能为空!!!');
    return;
  }
  else{
    if(password_register.value!=repassword.value){
      ElMessage.error('两次输入的密码不一致!!!');
      return;
    }
    else{
      const publicKeyPem = `-----BEGIN PUBLIC KEY-----
    MFwwDQYJKoZIhvcNAQEBBQADSwAwSAJBAKHP3hnFtL1g3bpgDMFAma1MofY9UmInthR8vK5Q9/dYcfdqvzLFRdRCPdeefqGO+BIFfLeCKJi4odn61XJEvp8CAwEAAQ==
    -----END PUBLIC KEY-----`
      const publicKey = forge.pki.publicKeyFromPem(publicKeyPem);
      const encrypted = forge.util.encode64(publicKey.encrypt(password_register.value));
      try{
        await registerAPI({username: username_register.value, email: email_register.value, password: encrypted, verification_code: verificationCode_register.value});
        ElMessage.success('注册成功！');
        active.value = false; // Reset active to false when registration is successful
      }catch(error){
        ElMessage.error('注册失败！');
        console.error(error);
      }
    }
  }
}

const showRegister = (event) => {
  event.preventDefault();
  active.value = true; // Set active to true when switching to register form
};

const showLogin = (event) => {
  event.preventDefault();
  active.value = false; // Set active to false when switching back to login form
};

// Mounted lifecycle hook for dynamically loading external scripts
onMounted(() => {
  if (timer) {
    clearInterval(timer);
  }

  const scriptModule = document.createElement('script');
  scriptModule.type = 'module';
  scriptModule.src = 'https://unpkg.com/ionicons@7.1.0/dist/ionicons/ionicons.esm.js';
  document.head.appendChild(scriptModule);

  const scriptNoModule = document.createElement('script');
  scriptNoModule.nomodule = true;
  scriptNoModule.src = 'https://unpkg.com/ionicons@7.1.0/dist/ionicons/ionicons.js';
  document.head.appendChild(scriptNoModule);
});
</script>

<style scoped>
/* Add your CSS styles here */
@import url('login.css');
</style>
