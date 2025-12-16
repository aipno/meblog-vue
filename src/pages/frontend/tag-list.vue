<template>
  <Header></Header>

  <!-- 主内容区域 -->
  <main class="container max-w-screen-xl mx-auto px-4 md:px-6 py-8 min-h-[80vh]">
    <!-- grid 表格布局，分为 12 列 -->
    <div class="grid grid-cols-1 lg:grid-cols-12 gap-8">

      <!-- 左边栏，占用 9 列 -->
      <div class="col-span-1 lg:col-span-9 space-y-6">
        <!-- 标签列表容器 -->
        <div
          class="w-full p-6 md:p-8 bg-white border border-gray-100 rounded-2xl shadow-sm dark:bg-gray-800 dark:border-gray-700 transition-all duration-300 hover:shadow-lg">

          <!-- 标签标题 -->
          <h2
            class="flex items-center mb-8 text-2xl font-bold text-gray-800 dark:text-white border-b border-gray-100 dark:border-gray-700 pb-4">
            <span class="p-2 mr-3 rounded-lg bg-gradient-to-br from-pink-500 to-rose-500 text-white shadow-md">
              <!-- 标签图标 -->
              <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path
                  d="M7 7h.01M7 3h5c.512 0 1.024.195 1.414.586l7 7a2 2 0 010 2.828l-7 7a2 2 0 01-2.828 0l-7-7A1.994 1.994 0 013 12V7a4 4 0 014-4z"
                  stroke-linecap="round" stroke-linejoin="round" stroke-width="2" />
              </svg>
            </span>
            全部标签
            <span
              class="ml-3 text-sm font-normal text-gray-500 dark:text-gray-400 bg-gray-100 dark:bg-gray-700 px-3 py-1 rounded-full">
              共 {{ tags.length }} 个
            </span>
          </h2>

          <!-- 加载状态 -->
          <div v-if="loading" class="flex flex-wrap gap-4 animate-pulse">
            <div v-for="n in 12" :key="n" class="h-10 w-24 bg-gray-200 dark:bg-gray-700 rounded-full"></div>
          </div>

          <!-- 标签列表 (Flex 布局) -->
          <div v-else class="flex flex-wrap gap-4">
            <router-link v-for="(tag, index) in tags" :key="index"
              :to="{ path: '/tag/article/list', query: { id: tag.id, name: tag.name } }"
              class="group relative inline-flex items-center px-5 py-2.5 text-sm font-medium rounded-full cursor-pointer transition-all duration-300 border select-none
                      bg-gray-50 text-gray-700 border-transparent
                      hover:bg-gradient-to-r hover:from-pink-500 hover:to-rose-500 hover:text-white hover:shadow-lg hover:shadow-pink-500/30 hover:-translate-y-1
                      focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-pink-500
                      dark:bg-gray-700/50 dark:text-gray-300 dark:hover:bg-gradient-to-r dark:hover:from-pink-600 dark:hover:to-rose-600 dark:hover:text-white">
              <span class="relative z-10 flex items-center">
                <span class="opacity-70 mr-1">#</span> {{ tag.name }}
              </span>

              <!-- 文章数徽章 -->
              <span class="ml-2.5 text-xs font-bold opacity-80 bg-white text-gray-600 px-2 py-0.5 rounded-full transition-colors
                           group-hover:bg-white/20 group-hover:text-white
                           dark:bg-gray-600 dark:text-gray-300">
                {{ tag.articlesTotal }}
              </span>
            </router-link>
          </div>

          <!-- 空状态 -->
          <div v-if="!loading && (!tags || tags.length === 0)" class="py-16 text-center">
            <div
              class="inline-flex items-center justify-center w-16 h-16 rounded-full bg-gray-100 dark:bg-gray-700 mb-4 text-gray-400">
              <svg class="w-8 h-8" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                  d="M7 21h10a2 2 0 002-2V9.414a1 1 0 00-.293-.707l-5.414-5.414A1 1 0 0012.586 2H7a2 2 0 00-2 2v14a2 2 0 002 2z">
                </path>
              </svg>
            </div>
            <p class="text-gray-500 dark:text-gray-400 text-lg">暂无标签数据</p>
          </div>
        </div>
      </div>

      <!-- 右边侧边栏，占用 3 列 -->
      <aside class="col-span-1 lg:col-span-3 space-y-6">
        <div class="sticky top-[5.5rem] space-y-6">
          <!-- 博主信息 -->
          <UserInfoCard class="animate__animated animate__fadeInRight" />

          <!-- 分类 (作为补充导航) -->
          <CategoryListCard class="animate__animated animate__fadeInRight animate__delay-200ms" />
        </div>
      </aside>
    </div>

  </main>

  <!-- 返回顶部 -->
  <ScrollToTopButton></ScrollToTopButton>

  <Footer></Footer>
</template>

<script setup>
import Header from '@/layouts/frontend/components/Header.vue'
import Footer from '@/layouts/frontend/components/Footer.vue'
import UserInfoCard from '@/layouts/frontend/components/UserInfoCard.vue'
import CategoryListCard from '@/layouts/frontend/components/CategoryListCard.vue'
import ScrollToTopButton from '@/layouts/frontend/components/ScrollToTopButton.vue'
import { getTagList } from '@/api/frontend/tag'
import { ref, onMounted } from 'vue'

// 加载状态
const loading = ref(true)
// 所有标签
const tags = ref([])

onMounted(() => {
  fetchTags()
})

const fetchTags = () => {
  loading.value = true
  getTagList({}).then((res) => {
    if (res.success) {
      tags.value = res.data
    }
  }).finally(() => {
    loading.value = false
  })
}
</script>

<style scoped>
/* 可以在这里补充一些特定的过渡效果样式 */
</style>