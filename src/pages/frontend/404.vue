<template>
  <div
      class="min-h-screen flex flex-col from-gray-50 to-gray-100 dark:from-[#0d1117] dark:to-[#1a1d23] transition-colors duration-300 font-sans">
    <Header></Header>

    <!-- 主内容区域 -->
    <main class="flex-grow flex flex-col items-center justify-center p-4 relative overflow-hidden">
      <!-- 背景装饰 -->
      <div class="absolute inset-0 overflow-hidden -z-10">
        <div
            class="absolute top-1/4 left-1/4 w-64 h-64 bg-blue-500/10 dark:bg-blue-500/5 rounded-full blur-3xl animate-pulse"></div>
        <div
            class="absolute bottom-1/3 right-1/4 w-72 h-72 bg-purple-500/10 dark:bg-purple-500/5 rounded-full blur-3xl animate-pulse delay-1000"></div>
      </div>

      <div class="w-full max-w-4xl flex flex-col items-center text-center z-10 py-8">

        <!-- 文字提示 -->
        <div class="animate__animated animate__fadeInUp animate__delay-1s relative z-10 mb-10">
          <h1 class="text-6xl md:text-7xl lg:text-8xl font-extrabold text-transparent bg-clip-text bg-gradient-to-r from-blue-600 via-purple-500 to-indigo-600 dark:from-blue-400 dark:via-purple-400 dark:to-indigo-400 mb-6 tracking-tight">
            404
          </h1>
          <h2 class="text-2xl md:text-3xl font-bold text-gray-800 dark:text-gray-200 mb-3">
            页面未找到
          </h2>
          <p class="text-lg md:text-xl text-gray-600 dark:text-gray-300 font-medium max-w-md mx-auto px-4">
            抱歉，您访问的页面似乎迷失在数字宇宙中了...
          </p>
        </div>

        <!-- 按钮组 -->
        <div
            class="mt-4 mb-16 animate__animated animate__fadeInUp animate__delay-2s relative z-10 flex flex-col sm:flex-row gap-4">
          <button
              class="group relative inline-flex items-center justify-center px-8 py-4 text-base font-semibold text-white transition-all duration-300 bg-gradient-to-r from-blue-600 to-indigo-700 font-pj rounded-full focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-600 hover:from-blue-700 hover:to-indigo-800 shadow-lg hover:shadow-blue-500/30 hover:-translate-y-1 transform"
              type="button"
              @click="goHome">
            <svg class="w-5 h-5 mr-2 transition-transform duration-300 group-hover:-translate-x-1" fill="none"
                 stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
              <path
                  d="M3 12l2-2m0 0l7-7 7 7M5 10v10a1 1 0 001 1h3m10-11l2 2m-2-2v10a1 1 0 01-1 1h-3m-6 0a1 1 0 001-1v-4a1 1 0 011-1h2a1 1 0 011 1v4a1 1 0 001 1m-6 0h6"
                  stroke-linecap="round" stroke-linejoin="round" stroke-width="2"></path>
            </svg>
            <span>返回首页</span>
          </button>

          <button
              class="group relative inline-flex items-center justify-center px-8 py-4 text-base font-semibold text-gray-700 dark:text-gray-200 transition-all duration-300 bg-white dark:bg-gray-800 border border-gray-300 dark:border-gray-600 font-pj rounded-full focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-gray-300 hover:bg-gray-50 dark:hover:bg-gray-700 shadow hover:shadow-gray-500/10 hover:-translate-y-1 transform"
              type="button"
              @click="goBack">
            <svg class="w-5 h-5 mr-2 transition-transform duration-300 group-hover:-translate-x-1" fill="none"
                 stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
              <path d="M10 19l-7-7m0 0l7-7m-7 7h18" stroke-linecap="round" stroke-linejoin="round"
                    stroke-width="2"></path>
            </svg>
            <span>返回上页</span>
          </button>
        </div>

        <!-- 倒计时提示 -->
        <div v-if="countdown > 0"
             class="animate__animated animate__fadeIn animate__delay-3s text-sm text-gray-500 dark:text-gray-400 mt-4">
          <p>{{ countdown }} 秒后自动返回首页</p>
        </div>
      </div>
    </main>

    <Footer class="mt-auto"></Footer>
  </div>
</template>

<script setup>
import Header from '@/layouts/frontend/components/Header.vue'
import Footer from '@/layouts/frontend/components/Footer.vue'
import {useRouter} from 'vue-router'
import {onBeforeUnmount, onMounted, ref} from 'vue'

const router = useRouter()
const countdown = ref(10)
let timer = null

const goHome = () => {
  router.push('/')
}

const goBack = () => {
  router.go(-1)
}

onMounted(() => {
  timer = setInterval(() => {
    countdown.value--
    if (countdown.value <= 0) {
      goHome()
    }
  }, 1000)
})

onBeforeUnmount(() => {
  if (timer) clearInterval(timer)
})
</script>

<style scoped>
/* 深色模式下的SVG调整 */
.dark svg {
  filter: brightness(0.9) contrast(1.1);
}

/* 添加一些额外的动画效果 */
@keyframes float {
  0% {
    transform: translateY(0px);
  }
  50% {
    transform: translateY(-10px);
  }
  100% {
    transform: translateY(0px);
  }
}

.animate-float {
  animation: float 3s ease-in-out infinite;
}
</style>
