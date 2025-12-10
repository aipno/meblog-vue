<template>
  <div class="min-h-screen flex items-center justify-center bg-gray-50 dark:bg-gray-900 transition-colors duration-300">

    <!-- 注册容器 -->
    <div
        class="flex w-full max-w-5xl h-full md:h-[650px] bg-white dark:bg-gray-800 rounded-2xl shadow-2xl overflow-hidden m-4 transition-colors duration-300">

      <!-- 左侧：品牌展示区 (仅在大屏显示) -->
      <div
          class="hidden md:flex md:w-1/2 bg-gradient-to-br from-indigo-600 to-sky-500 relative items-center justify-center p-12 overflow-hidden">
        <!-- 背景装饰 -->
        <div class="absolute top-10 left-10 w-40 h-40 bg-white/10 rounded-full blur-3xl animate-pulse"></div>
        <div class="absolute bottom-10 right-10 w-60 h-60 bg-white/10 rounded-full blur-3xl"></div>

        <div class="relative z-10 text-center text-white">
          <h2 class="text-4xl font-extrabold mb-6 tracking-tight animate__animated animate__fadeInDown">加入 MeBlog</h2>
          <p class="text-indigo-100 text-lg mb-8 animate__animated animate__fadeInUp animate__delay-1s">
            开启您的创作之旅，记录生活，分享技术。
          </p>
          <!-- 插画 -->
          <div class="relative w-72 h-72 mx-auto animate__animated animate__fadeInUp animate__delay-1s">
            <img alt="Register Illustration"
                 class="w-full h-full object-contain drop-shadow-xl transform hover:scale-105 transition-transform duration-500"
                 src="@/assets/image/PwnnyBrigade.png"/>
          </div>
        </div>
      </div>

      <!-- 右侧：注册表单区 -->
      <div class="w-full md:w-1/2 p-8 md:p-12 flex flex-col justify-center relative bg-white dark:bg-gray-800">

        <!-- 顶部返回登录 -->
        <div class="absolute top-6 right-6 text-sm text-gray-500 dark:text-gray-400">
          已有账号？
          <span class="text-sky-600 hover:text-sky-500 font-bold cursor-pointer hover:underline transition-colors"
                @click="goToLogin">立即登录</span>
        </div>

        <div class="max-w-md mx-auto w-full">
          <div class="text-center mb-8">
            <h1 class="text-3xl font-bold text-gray-900 dark:text-white mb-2">创建账号 🚀</h1>
            <p class="text-gray-500 dark:text-gray-400 text-sm">填写以下信息完成注册</p>
          </div>

          <el-form ref="formRef" :model="registerForm" :rules="registerRules" class="space-y-5" size="large"
                   @submit.prevent>

            <el-form-item prop="username">
              <el-input
                  v-model="registerForm.username"
                  :prefix-icon="User"
                  class="h-11"
                  placeholder="用户名"
              />
            </el-form-item>

            <el-form-item prop="password">
              <el-input
                  v-model="registerForm.password"
                  :prefix-icon="Lock"
                  class="h-11"
                  placeholder="设置密码"
                  show-password
                  type="password"
              />
            </el-form-item>

            <el-form-item prop="confirmPassword">
              <el-input
                  v-model="registerForm.confirmPassword"
                  :prefix-icon="Lock"
                  class="h-11"
                  placeholder="确认密码"
                  show-password
                  type="password"
              />
            </el-form-item>

            <el-form-item prop="email">
              <el-input
                  v-model="registerForm.email"
                  :prefix-icon="Message"
                  class="h-11"
                  placeholder="电子邮箱"
              />
            </el-form-item>

            <el-form-item prop="code">
              <div class="flex gap-3 w-full">
                <el-input
                    v-model="registerForm.code"
                    :prefix-icon="Key"
                    class="h-11 flex-1"
                    placeholder="验证码"
                />
                <el-button
                    :disabled="countdown > 0"
                    class="h-11 w-32 font-medium !rounded-xl !border-sky-100 text-sky-600 hover:!bg-sky-50 hover:!text-sky-700 dark:!bg-gray-700 dark:!border-gray-600 dark:text-sky-400"
                    @click="getCode"
                >
                  {{ countdown > 0 ? `${countdown}s` : '获取验证码' }}
                </el-button>
              </div>
            </el-form-item>

            <div class="pt-2">
              <el-button
                  :loading="loading"
                  class="w-full h-12 text-base font-bold tracking-wide bg-gradient-to-r from-indigo-600 to-sky-500 hover:from-indigo-700 hover:to-sky-600 border-none transition-all duration-300 shadow-lg hover:shadow-indigo-500/30 !rounded-xl"
                  type="primary"
                  @click="handleRegister"
              >
                注 册
              </el-button>
            </div>

            <!-- 辅助按钮 -->
            <div class="text-center mt-4">
              <el-button class="text-gray-400 hover:text-gray-600 dark:hover:text-gray-300" link @click="resetForm">
                重置表单
              </el-button>
            </div>
          </el-form>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import {onBeforeUnmount, onMounted, reactive, ref} from 'vue'
import {showMessage} from '@/composables/utils'
import {useRouter} from 'vue-router'
import {getEmailCode} from '@/api/admin/email'
import {register} from '@/api/admin/user'
import {Key, Lock, Message, User} from '@element-plus/icons-vue' // 引入图标

const router = useRouter()
const formRef = ref(null)
const loading = ref(false)

const registerForm = reactive({
  username: '',
  password: '',
  confirmPassword: '',
  email: '',
  code: '',
})

// 验证规则 (保持原有逻辑)
const registerRules = {
  username: [
    {required: true, message: '请输入用户名', trigger: 'blur'},
    {min: 3, max: 10, message: '用户名长度必须在 3 到 10 个字符', trigger: 'blur'},
  ],
  password: [
    {required: true, message: '请输入密码', trigger: 'blur'},
    {min: 6, max: 12, message: '密码长度必须在 6 到 12 个字符', trigger: 'blur'},
  ],
  confirmPassword: [
    {required: true, message: '请确认密码', trigger: 'blur'},
    {min: 6, max: 12, message: '密码长度必须在 6 到 12 个字符', trigger: 'blur'},
    {
      validator: (rule, value, callback) => {
        if (value !== registerForm.password) {
          callback(new Error('两次输入密码不一致'))
        } else {
          callback()
        }
      },
      trigger: 'blur'
    }
  ],
  email: [
    {required: true, message: '请输入邮箱', trigger: 'blur'},
    {type: 'email', message: '请输入正确的邮箱格式', trigger: 'blur'}
  ],
  code: [
    {required: true, message: '请输入验证码', trigger: 'blur'}
  ],
}

const countdown = ref(0);
const timer = ref(null);

// 回车提交支持
function onKeyUp(e) {
  if (e.key === 'Enter') {
    handleRegister()
  }
}

onMounted(() => {
  document.addEventListener('keyup', onKeyUp)
})

// 获取验证码
const getCode = () => {
  if (!registerForm.email) {
    showMessage('请先填写邮箱地址', 'warning')
    return
  }

  // 简单校验邮箱格式
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
  if (!emailRegex.test(registerForm.email)) {
    showMessage('邮箱格式不正确', 'warning')
    return
  }

  getEmailCode({email: registerForm.email}).then(res => {
    if (res.success === true) {
      showMessage('验证码已发送')
      startCountdown()
    } else {
      showMessage(res.message || '验证码发送失败', 'error')
    }
  }).catch(err => {
    showMessage('验证码发送失败', 'error')
  })
}

const startCountdown = () => {
  countdown.value = 60;
  timer.value = setInterval(() => {
    countdown.value -= 1;
    if (countdown.value <= 0) {
      clearInterval(timer.value);
      countdown.value = 0;
    }
  }, 1000);
}

const handleRegister = () => {
  formRef.value.validate((valid) => {
    if (valid) {
      loading.value = true
      register(registerForm).then(res => {
        if (res.success === true) {
          showMessage('注册成功！')
          router.push('/admin/login')
        } else {
          showMessage(res.message || '注册失败', 'error')
        }
      }).catch(err => {
        showMessage('注册失败', 'error')
      }).finally(() => {
        loading.value = false
      })
    }
  })
}

const resetForm = () => {
  formRef.value.resetFields()
}

const goToLogin = () => {
  router.push('/admin/login')
}

onBeforeUnmount(() => {
  document.removeEventListener('keyup', onKeyUp)
  if (timer.value) clearInterval(timer.value);
});
</script>

<style scoped>
/* 深度选择器定制 Element Plus 组件样式 */
:deep(.el-input__wrapper) {
  background-color: #f9fafb; /* bg-gray-50 */
  border-radius: 0.75rem; /* rounded-xl */
  box-shadow: none !important;
  border: 1px solid #e5e7eb; /* border-gray-200 */
  transition: all 0.3s;
}

:deep(.el-input__wrapper.is-focus) {
  background-color: #fff;
  border-color: #6366f1; /* indigo-500 */
  box-shadow: 0 0 0 1px #6366f1 !important;
}

/* 暗黑模式适配 */
.dark :deep(.el-input__wrapper) {
  background-color: #374151; /* bg-gray-700 */
  border-color: #4b5563; /* border-gray-600 */
}

.dark :deep(.el-input__inner) {
  color: #fff;
}
</style>