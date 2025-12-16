<template>
  <Header></Header>

  <!-- 主内容区域 -->
  <main class="container max-w-screen-xl mx-auto px-4 md:px-6 py-12 min-h-[80vh]">

    <!-- 页面标题头 -->
    <div class="mb-16 text-center animate__animated animate__fadeInDown">
      <h1
        class="text-4xl md:text-5xl font-extrabold text-gray-900 dark:text-white tracking-tight mb-6 relative inline-block">
        <span class="bg-clip-text text-transparent bg-gradient-to-r from-sky-500 to-indigo-500">知识库</span>
        <span
          class="absolute -bottom-2 left-0 w-full h-1.5 bg-gradient-to-r from-sky-500 to-indigo-500 rounded-full opacity-30"></span>
      </h1>
      <p class="text-gray-500 dark:text-gray-400 text-lg max-w-2xl mx-auto leading-relaxed">
        体系化的知识沉淀，构建完整的技术图谱。在这里，您可以找到系列教程、文档手册以及深度的技术专栏。
      </p>
    </div>

    <!-- 加载状态：骨架屏 -->
    <div v-if="loading" class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-8">
      <div v-for="n in 6" :key="n"
        class="bg-white dark:bg-gray-800 rounded-2xl overflow-hidden shadow-sm border border-gray-100 dark:border-gray-700 h-[420px] animate-pulse">
        <!-- 封面骨架 -->
        <div class="w-full h-52 bg-gray-200 dark:bg-gray-700"></div>
        <!-- 内容骨架 -->
        <div class="p-6 space-y-4">
          <div class="h-6 bg-gray-200 dark:bg-gray-700 rounded w-3/4"></div>
          <div class="space-y-2">
            <div class="h-4 bg-gray-200 dark:bg-gray-700 rounded w-full"></div>
            <div class="h-4 bg-gray-200 dark:bg-gray-700 rounded w-5/6"></div>
            <div class="h-4 bg-gray-200 dark:bg-gray-700 rounded w-2/3"></div>
          </div>
          <div class="pt-4 flex justify-between items-center">
            <div class="h-4 bg-gray-200 dark:bg-gray-700 rounded w-20"></div>
            <div class="h-8 w-8 bg-gray-200 dark:bg-gray-700 rounded-full"></div>
          </div>
        </div>
      </div>
    </div>

    <!-- 知识库列表 -->
    <div v-else-if="wikis && wikis.length > 0" class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-8">
      <router-link v-for="(wiki, index) in wikis" :key="index"
        :to="{ path: '/wiki/' + wiki.id, query: { articleId: wiki.firstArticleId } }"
        :style="{ animationDelay: `${index * 0.1}s` }"
        class="group flex flex-col bg-white dark:bg-gray-800 border border-gray-100 dark:border-gray-700 rounded-2xl overflow-hidden shadow-sm hover:shadow-2xl hover:shadow-sky-500/10 hover:-translate-y-2 transition-all duration-300 animate__animated animate__fadeInUp focus:outline-none focus:ring-4 focus:ring-sky-500/30">

        <!-- 封面图 -->
        <div class="relative w-full h-52 overflow-hidden bg-gray-100 dark:bg-gray-700">
          <img :src="wiki.cover" :alt="wiki.title + ' 封面'"
            class="w-full h-full object-cover transition-transform duration-700 group-hover:scale-110" loading="lazy" />

          <!-- 遮罩 -->
          <div
            class="absolute inset-0 bg-gradient-to-t from-black/60 via-transparent to-transparent opacity-60 group-hover:opacity-80 transition-opacity duration-300">
          </div>

          <!-- 置顶徽章 -->
          <div v-if="wiki.isTop"
            class="absolute top-4 right-4 bg-rose-500 text-white text-xs font-bold px-3 py-1 rounded-full shadow-lg flex items-center gap-1.5 z-10 backdrop-blur-md bg-opacity-90">
            <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path d="M5 10l7-7m0 0l7 7m-7-7v18" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" />
            </svg>
            <span>置顶</span>
          </div>
        </div>

        <!-- 内容区 -->
        <div class="flex-1 p-7 flex flex-col relative">
          <!-- 标题 -->
          <h2
            class="text-xl font-bold text-gray-900 dark:text-white mb-3 line-clamp-1 group-hover:text-sky-600 dark:group-hover:text-sky-400 transition-colors">
            {{ wiki.title }}
          </h2>

          <!-- 摘要 -->
          <p class="text-sm text-gray-500 dark:text-gray-400 line-clamp-3 mb-6 flex-grow leading-relaxed">
            {{ wiki.summary || '暂无简介' }}
          </p>

          <!-- 底部操作栏 -->
          <div class="mt-auto pt-5 border-t border-gray-100 dark:border-gray-700 flex items-center justify-between">
            <span
              class="text-xs font-semibold uppercase tracking-wider text-gray-400 group-hover:text-sky-600 dark:group-hover:text-sky-400 transition-colors flex items-center gap-1">
              <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                  d="M12 6.253v13m0-13C10.832 5.477 9.246 5 7.5 5S4.168 5.477 3 6.253v13C4.168 18.477 5.754 18 7.5 18s3.332.477 4.5 1.253m0-13C13.168 5.477 14.754 5 16.5 5c1.747 0 3.332.477 4.5 1.253v13C19.832 18.477 18.247 18 16.5 18c-1.746 0-3.332.477-4.5 1.253">
                </path>
              </svg>
              开始阅读
            </span>
            <div
              class="w-9 h-9 rounded-full bg-gray-50 dark:bg-gray-700 text-gray-400 group-hover:bg-sky-500 group-hover:text-white transition-all duration-300 flex items-center justify-center transform group-hover:rotate-45">
              <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path d="M14 5l7 7m0 0l-7 7m7-7H3" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" />
              </svg>
            </div>
          </div>
        </div>
      </router-link>
    </div>

    <!-- 空状态 -->
    <div v-else
      class="py-24 text-center bg-white dark:bg-gray-800 rounded-2xl border border-gray-100 dark:border-gray-700 shadow-sm animate__animated animate__fadeIn">
      <div
        class="inline-flex items-center justify-center w-20 h-20 rounded-full bg-gray-100 dark:bg-gray-700 mb-6 text-gray-400">
        <svg class="w-10 h-10" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
            d="M12 6.253v13m0-13C10.832 5.477 9.246 5 7.5 5S4.168 5.477 3 6.253v13C4.168 18.477 5.754 18 7.5 18s3.332.477 4.5 1.253m0-13C13.168 5.477 14.754 5 16.5 5c1.747 0 3.332.477 4.5 1.253v13C19.832 18.477 18.247 18 16.5 18c-1.746 0-3.332.477-4.5 1.253">
          </path>
        </svg>
      </div>
      <h3 class="text-xl font-bold text-gray-900 dark:text-white mb-2">暂无知识库</h3>
      <p class="text-gray-500 dark:text-gray-400 max-w-sm mx-auto">博主正在精心整理知识体系，精彩内容即将呈现，敬请期待...</p>
    </div>

  </main>

  <!-- 返回顶部 -->
  <ScrollToTopButton></ScrollToTopButton>

  <Footer></Footer>
</template>

<script setup>
import Header from '@/layouts/frontend/components/Header.vue'
import Footer from '@/layouts/frontend/components/Footer.vue'
import ScrollToTopButton from '@/layouts/frontend/components/ScrollToTopButton.vue'
import { getWikiList } from "@/api/frontend/wiki.js";
import { ref, onMounted } from "vue";
import { useRouter } from 'vue-router'
import { showMessage } from "@/composables/utils.js";

const router = useRouter()
// 知识库
const wikis = ref([])
// 加载状态
const loading = ref(true)

onMounted(() => {
  fetchWikiList()
})

const fetchWikiList = () => {
  loading.value = true
  getWikiList().then(res => {
    if (res.success) {
      wikis.value = res.data
    } else {
      showMessage(res.message, 'error')
    }
  }).finally(() => {
    loading.value = false
  })
}
</script>

<style scoped>
/* 可以在这里补充一些特定的过渡效果样式 */
</style>