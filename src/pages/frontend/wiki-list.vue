<template>
  <Header></Header>

  <!-- 主内容区域 -->
  <main class="container max-w-screen-xl mx-auto px-4 md:px-6 py-10 min-h-[80vh]">

    <!-- 页面标题头 -->
    <div class="mb-12 text-center animate__animated animate__fadeInDown">
      <h1 class="text-3xl md:text-4xl font-extrabold text-gray-900 dark:text-white tracking-tight mb-4">
        知识库
      </h1>
      <p class="text-gray-500 dark:text-gray-400 text-sm md:text-base max-w-2xl mx-auto">
        体系化的知识沉淀，构建完整的技术图谱。在这里，您可以找到系列教程、文档手册以及深度的技术专栏。
      </p>
    </div>

    <!-- 知识库列表 -->
    <div v-if="wikis && wikis.length > 0" class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-8">
      <div v-for="(wiki, index) in wikis" :key="index"
           :style="{ animationDelay: `${index * 0.1}s` }"
           class="group flex flex-col bg-white dark:bg-gray-800 border border-gray-100 dark:border-gray-700 rounded-2xl overflow-hidden shadow-sm hover:shadow-xl hover:-translate-y-1 transition-all duration-300 animate__animated animate__fadeInUp cursor-pointer"
           @click="goWikiArticleDetailPage(wiki.id, wiki.firstArticleId)">

        <!-- 封面图 -->
        <div class="relative w-full h-48 overflow-hidden bg-gray-100 dark:bg-gray-700">
          <img :src="wiki.cover"
               alt="cover" class="w-full h-full object-cover transition-transform duration-500 group-hover:scale-110"
               loading="lazy"/>

          <!-- 遮罩 -->
          <div
              class="absolute inset-0 bg-black/10 opacity-0 group-hover:opacity-100 transition-opacity duration-300"></div>

          <!-- 置顶徽章 -->
          <div v-if="wiki.isTop"
               class="absolute top-3 right-3 bg-red-500/90 backdrop-blur-sm text-white text-xs font-bold px-2.5 py-1 rounded-full shadow-sm flex items-center gap-1 z-10">
            <svg class="w-3 h-3" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path d="M5 10l7-7m0 0l7 7m-7-7v18" stroke-linecap="round" stroke-linejoin="round" stroke-width="2"/>
            </svg>
            置顶
          </div>
        </div>

        <!-- 内容区 -->
        <div class="flex-1 p-6 flex flex-col">
          <!-- 标题 -->
          <h2 class="text-xl font-bold text-gray-800 dark:text-white mb-3 line-clamp-1 group-hover:text-sky-600 dark:group-hover:text-sky-400 transition-colors">
            {{ wiki.title }}
          </h2>

          <!-- 摘要 -->
          <p class="text-sm text-gray-500 dark:text-gray-400 line-clamp-3 mb-6 flex-grow leading-relaxed">
            {{ wiki.summary || '暂无简介' }}
          </p>

          <!-- 底部操作栏 -->
          <div class="mt-auto pt-4 border-t border-gray-100 dark:border-gray-700 flex items-center justify-between">
             <span class="text-xs text-gray-400 font-medium group-hover:text-sky-500 transition-colors">
                 开始阅读
             </span>
            <button
                class="w-8 h-8 rounded-full bg-gray-50 dark:bg-gray-700 text-gray-400 group-hover:bg-sky-500 group-hover:text-white transition-colors flex items-center justify-center">
              <svg class="w-4 h-4 transition-transform group-hover:translate-x-0.5" fill="none" stroke="currentColor"
                   viewBox="0 0 24 24">
                <path d="M14 5l7 7m0 0l-7 7m7-7H3" stroke-linecap="round" stroke-linejoin="round" stroke-width="2"/>
              </svg>
            </button>
          </div>
        </div>
      </div>
    </div>

    <!-- 空状态 -->
    <div v-else
         class="py-20 text-center bg-white dark:bg-gray-800 rounded-2xl border border-gray-100 dark:border-gray-700 animate__animated animate__fadeIn">
      <div class="w-64 h-64 mx-auto mb-6 opacity-80">
        <!-- 简单的 SVG 占位 -->
        <svg class="w-full h-full text-gray-200 dark:text-gray-700 fill-current" viewBox="0 0 400 400"
             xmlns="http://www.w3.org/2000/svg">
          <circle cx="200" cy="200" fill="currentColor" opacity="0.2" r="80"/>
          <path d="M160 180 H240 M160 220 H240" stroke="currentColor" stroke-linecap="round" stroke-width="10"/>
        </svg>
      </div>
      <h3 class="text-lg font-medium text-gray-900 dark:text-white">暂无知识库</h3>
      <p class="text-gray-500 dark:text-gray-400 mt-2 text-sm">博主正在努力整理知识体系，敬请期待...</p>
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
import {getWikiList} from "@/api/frontend/wiki.js";
import {ref} from "vue";
import {useRouter} from 'vue-router'
import {showMessage} from "@/composables/utils.js";

const router = useRouter()
// 知识库
const wikis = ref([])

getWikiList().then(res => {
  if (res.success) {
    wikis.value = res.data
  } else {
    showMessage(res.message, 'error')
  }
})

// 跳转文章详情页
const goWikiArticleDetailPage = (wikiId, articleId) => {
  // 如果知识库没有文章 (firstArticleId 为空)，则跳转到知识库详情页（可能显示空状态或目录）
  // 这里暂时保持原逻辑，如果 articleId 为空，路由参数可能需要处理
  router.push({path: '/wiki/' + wikiId, query: {articleId}})
}
</script>

<style scoped>
/* 可以在这里补充一些特定的过渡效果样式 */
</style>