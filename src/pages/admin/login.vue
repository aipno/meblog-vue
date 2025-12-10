<template>
  <div class="min-h-screen flex items-center justify-center bg-gray-50 dark:bg-gray-900 transition-colors duration-300">

    <!-- 登录容器 -->
    <div
        class="flex w-full max-w-5xl h-full md:h-[600px] bg-white dark:bg-gray-800 rounded-2xl shadow-2xl overflow-hidden m-4 transition-colors duration-300">

      <!-- 左侧：品牌展示区 (仅在大屏显示) -->
      <div
          class="hidden md:flex md:w-1/2 bg-gradient-to-br from-sky-500 to-indigo-600 relative items-center justify-center p-12 overflow-hidden">
        <!-- 背景装饰圆 -->
        <div
            class="absolute top-0 left-0 w-64 h-64 bg-white/10 rounded-full -translate-x-1/2 -translate-y-1/2 blur-3xl"></div>
        <div
            class="absolute bottom-0 right-0 w-80 h-80 bg-white/10 rounded-full translate-x-1/3 translate-y-1/3 blur-3xl"></div>

        <div class="relative z-10 text-center text-white">
          <h2 class="text-4xl font-extrabold mb-4 tracking-tight animate__animated animate__fadeInDown">MeBlog
            Admin</h2>
          <p class="text-sky-100 text-lg mb-8 animate__animated animate__fadeInUp animate__delay-1s">
            一款由 Spring Boot + Mybaits Plus + Vue 3.2 + Vite 4 开发的前后端分离博客。
          </p>
          <!-- 插画/图片 -->
          <div class="relative w-64 h-64 mx-auto animate__animated animate__zoomIn animate__delay-1s">
            <!-- 使用更通用的安全图标替代原图，或者保留原图 -->
            <img alt="Login Illustration"
                 class="w-full h-full object-contain drop-shadow-lg transform hover:scale-105 transition-transform duration-500"
                 src="@/assets/image/安全.png"/>
          </div>
        </div>
      </div>

      <!-- 右侧：登录表单区 -->
      <div class="w-full md:w-1/2 p-8 md:p-12 flex flex-col justify-center relative">

        <!-- 顶部操作栏 (暗黑模式切换) -->
        <div class="absolute top-6 right-6">
          <button
              class="p-2 rounded-lg text-gray-500 hover:bg-gray-100 dark:text-gray-400 dark:hover:bg-gray-700 transition-colors focus:outline-none"
              title="切换主题"
              @click="toggleDark()"
          >
            <svg v-if="!isDark" class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path
                  d="M12 3v1m0 16v1m9-9h-1M4 12H3m15.364 6.364l-.707-.707M6.343 6.343l-.707-.707m12.728 0l-.707.707M6.343 17.657l-.707.707M16 12a4 4 0 11-8 0 4 4 0 018 0z"
                  stroke-linecap="round" stroke-linejoin="round"
                  stroke-width="2"/>
            </svg>
            <svg v-else class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path d="M20.354 15.354A9 9 0 018.646 3.646 9.003 9.003 0 0012 21a9.003 9.003 0 008.354-5.646z"
                    stroke-linecap="round" stroke-linejoin="round"
                    stroke-width="2"/>
            </svg>
          </button>
        </div>

        <div class="max-w-md mx-auto w-full">
          <div class="text-center mb-10">
            <h1 class="text-3xl font-bold text-gray-900 dark:text-white mb-2">欢迎回来 👋</h1>
            <p class="text-gray-500 dark:text-gray-400">请输入您的账号密码进行登录</p>
          </div>

          <el-form ref="formRef" :model="form" :rules="rules" class="space-y-6" size="large" @submit.prevent>
            <el-form-item prop="username">
              <el-input
                  v-model="form.username"
                  :prefix-icon="User"
                  class="h-12 text-base"
                  placeholder="用户名"
              />
            </el-form-item>

            <el-form-item prop="password">
              <el-input
                  v-model="form.password"
                  :prefix-icon="Lock"
                  class="h-12 text-base"
                  placeholder="密码"
                  show-password
                  type="password"
              />
            </el-form-item>

            <div class="flex items-center justify-between text-sm">
              <el-checkbox class="!mr-0" label="记住我"/>
              <a class="text-sky-600 hover:text-sky-500 font-medium transition-colors" href="#">忘记密码?</a>
            </div>

            <el-button
                :loading="loading"
                class="w-full h-12 text-base font-semibold tracking-wide bg-gradient-to-r from-sky-500 to-indigo-600 hover:from-sky-600 hover:to-indigo-700 border-none transition-all duration-300 shadow-lg hover:shadow-sky-500/30"
                type="primary"
                @click="onSubmit"
            >
              登 录
            </el-button>
          </el-form>

          <p class="mt-8 text-center text-sm text-gray-500 dark:text-gray-400">
            还没有账号？
            <router-link class="text-sky-600 hover:text-sky-500 font-medium hover:underline transition-colors"
                         to="/register">
              立即注册
            </router-link>
          </p>
        </div>
      </div>
    </div>

  </div>
</template>

<script setup>
import {onBeforeUnmount, onMounted, reactive, ref} from 'vue';
import {Lock, User} from '@element-plus/icons-vue' // 导入图标
import {login} from '@/api/admin/user.js';
import {useRouter} from 'vue-router';
import {showMessage} from '@/composables/utils.js';
import {setToken} from "@/composables/cookie.js";
import {useUserStore} from '@/stores/user'
import {useDark, useToggle} from '@vueuse/core'
import 'element-plus/theme-chalk/dark/css-vars.css'

const router = useRouter();
const formRef = ref(null);
const loading = ref(false);
const userStore = useUserStore()

// 暗黑模式控制
const isDark = useDark()
const toggleDark = useToggle(isDark)

const rules = {
  username: [{required: true, message: '用户名不能为空', trigger: 'blur'}],
  password: [{required: true, message: '密码不能为空', trigger: 'blur'}],
};

function onKeyUp(e) {
  if (e.key === 'Enter') {
    onSubmit()
  }
}

onMounted(() => {
  document.addEventListener('keyup', onKeyUp)
})

onBeforeUnmount(() => {
  document.removeEventListener('keyup', onKeyUp)
})

const form = reactive({
  username: '',
  password: '',
});

const onSubmit = () => {
  formRef.value.validate((valid) => {
    if (!valid) return false

    loading.value = true;
    login(form.username, form.password).then((res) => {
      if (res.success === true) {
        showMessage('登录成功')
        setToken(res.data.token)
        userStore.setUserInfo()
        router.push('/admin/index')
      } else {
        let message = res.message;
        showMessage(message, 'error')
      }
    }).finally(() => {
      loading.value = false;
    })
  })
}
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
  border-color: var(--el-color-primary);
  box-shadow: 0 0 0 1px var(--el-color-primary) !important;
}

/* 暗黑模式适配 */
.dark :deep(.el-input__wrapper) {
  background-color: #374151; /* bg-gray-700 */
  border-color: #4b5563; /* border-gray-600 */
}

.dark :deep(.el-input__inner) {
  color: #fff;
}

/* 按钮样式微调 */
:deep(.el-button) {
  border-radius: 0.75rem;
}
</style>