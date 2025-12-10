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
            class="w-full p-8 bg-white border border-gray-100 rounded-2xl shadow-sm dark:bg-gray-800 dark:border-gray-700 animate__animated animate__fadeInLeft">

          <!-- 标签标题 -->
          <h2 class="flex items-center mb-6 text-xl font-bold text-gray-800 dark:text-white">
            <span class="p-2 mr-3 rounded-lg bg-pink-50 text-pink-600 dark:bg-pink-900/30 dark:text-pink-400">
               <!-- 标签图标 -->
               <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path
                   d="M7 7h.01M7 3h5c.512 0 1.024.195 1.414.586l7 7a2 2 0 010 2.828l-7 7a2 2 0 01-2.828 0l-7-7A1.994 1.994 0 013 12V7a4 4 0 014-4z"
                   stroke-linecap="round"
                   stroke-linejoin="round"
                   stroke-width="2"/></svg>
            </span>
            全部标签
            <span
                class="ml-3 text-sm font-normal text-gray-500 dark:text-gray-400 bg-gray-100 dark:bg-gray-700 px-2 py-1 rounded-md">
              共 {{ tags.length }} 个
            </span>
          </h2>

          <!-- 标签列表 (Flex 布局) -->
          <div class="flex flex-wrap gap-4">
            <a v-for="(tag, index) in tags" :key="index"
               class="group relative inline-flex items-center px-4 py-2 text-sm font-medium rounded-full cursor-pointer transition-all duration-300 border select-none
                      bg-white text-gray-600 border-gray-200
                      hover:border-pink-300 hover:text-pink-600 hover:bg-pink-50 hover:shadow-md hover:-translate-y-0.5
                      dark:bg-gray-700 dark:border-gray-600 dark:text-gray-300 dark:hover:bg-gray-600 dark:hover:text-white"
               @click="goTagArticleListPage(tag.id, tag.name)">

              <span># {{ tag.name }}</span>

              <!-- 文章数徽章 -->
              <span class="ml-2 text-xs opacity-80 bg-gray-100 text-gray-500 px-1.5 py-0.5 rounded-full transition-colors
                           group-hover:bg-pink-200 group-hover:text-pink-700
                           dark:bg-gray-600 dark:text-gray-400 dark:group-hover:bg-pink-900 dark:group-hover:text-pink-300">
                  {{ tag.articlesTotal }}
              </span>
            </a>
          </div>

          <!-- 空状态 -->
          <div v-if="!tags || tags.length === 0" class="py-10 text-center text-gray-400">
            暂无标签数据
          </div>
        </div>
      </div>

      <!-- 右边侧边栏，占用 3 列 -->
      <aside class="col-span-1 lg:col-span-3 space-y-6">
        <div class="sticky top-[5.5rem] space-y-6">
          <!-- 博主信息 -->
          <UserInfoCard class="animate__animated animate__fadeInRight"/>

          <!-- 分类 (作为补充导航) -->
          <CategoryListCard class="animate__animated animate__fadeInRight animate__delay-200ms"/>
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
import {getTagList} from '@/api/frontend/tag'
import {ref} from 'vue'
import {useRouter} from 'vue-router'

const router = useRouter()

// 跳转标签文章列表页
const goTagArticleListPage = (id, name) => {
  // 跳转时通过 query 携带参数（标签 ID、标签名称）
  router.push({path: '/tag/article/list', query: {id, name}})
}

// 所有标签
const tags = ref([])
getTagList({}).then((res) => {
  if (res.success) {
    tags.value = res.data
  }
})
</script>

<style scoped>
/* 可以在这里补充一些特定的过渡效果样式 */
</style>