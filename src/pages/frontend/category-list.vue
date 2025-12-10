<template>
  <Header></Header>

  <!-- 主内容区域 -->
  <main class="container max-w-screen-xl mx-auto px-4 md:px-6 py-8 min-h-[80vh]">
    <!-- grid 表格布局，分为 12 列 -->
    <div class="grid grid-cols-1 lg:grid-cols-12 gap-8">

      <!-- 左边栏，占用 9 列 -->
      <div class="col-span-1 lg:col-span-9 space-y-6">
        <!-- 分类列表容器 -->
        <div
            class="w-full p-8 bg-white border border-gray-100 rounded-2xl shadow-sm dark:bg-gray-800 dark:border-gray-700 animate__animated animate__fadeInLeft">

          <!-- 分类标题 -->
          <h2 class="flex items-center mb-6 text-xl font-bold text-gray-800 dark:text-white">
            <span class="p-2 mr-3 rounded-lg bg-indigo-50 text-indigo-600 dark:bg-indigo-900/30 dark:text-indigo-400">
               <!-- 文件夹图标 -->
               <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path
                   d="M19 11H5m14 0a2 2 0 012 2v6a2 2 0 01-2 2H5a2 2 0 01-2-2v-6a2 2 0 012-2m14 0V9a2 2 0 00-2-2M5 11V9a2 2 0 012-2m0 0V5a2 2 0 012-2h6a2 2 0 012 2v2M7 7h10"
                   stroke-linecap="round"
                   stroke-linejoin="round"
                   stroke-width="2"/></svg>
            </span>
            全部分类
            <span
                class="ml-3 text-sm font-normal text-gray-500 dark:text-gray-400 bg-gray-100 dark:bg-gray-700 px-2 py-1 rounded-md">
              共 {{ categories.length }} 个
            </span>
          </h2>

          <!-- 分类列表 (Flex 布局) -->
          <div class="flex flex-wrap gap-4">
            <a v-for="(category, index) in categories" :key="index"
               class="group relative inline-flex items-center px-5 py-2.5 text-base font-medium rounded-full cursor-pointer transition-all duration-300 border select-none
                      bg-white text-gray-600 border-gray-200
                      hover:border-sky-300 hover:text-sky-600 hover:bg-sky-50 hover:shadow-md hover:-translate-y-0.5
                      dark:bg-gray-700 dark:border-gray-600 dark:text-gray-300 dark:hover:bg-gray-600 dark:hover:text-white"
               @click="goCategoryArticleListPage(category.id, category.name)">

              <span>{{ category.name }}</span>

              <!-- 文章数徽章 -->
              <span class="ml-3 text-xs opacity-80 bg-gray-100 text-gray-500 px-2 py-0.5 rounded-full transition-colors
                           group-hover:bg-sky-200 group-hover:text-sky-700
                           dark:bg-gray-600 dark:text-gray-400 dark:group-hover:bg-sky-900 dark:group-hover:text-sky-300">
                  {{ category.articlesTotal }}
              </span>
            </a>
          </div>

          <!-- 空状态 -->
          <div v-if="!categories || categories.length === 0" class="py-10 text-center text-gray-400">
            暂无分类数据
          </div>
        </div>
      </div>

      <!-- 右边侧边栏，占用 3 列 -->
      <aside class="col-span-1 lg:col-span-3 space-y-6">
        <div class="sticky top-[5.5rem] space-y-6">
          <!-- 博主信息 -->
          <UserInfoCard class="animate__animated animate__fadeInRight"/>

          <!-- 标签 (虽然这是分类页，但展示标签云也是合理的导航补充) -->
          <TagListCard class="animate__animated animate__fadeInRight animate__delay-200ms"/>
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
import TagListCard from '@/layouts/frontend/components/TagListCard.vue'
import ScrollToTopButton from '@/layouts/frontend/components/ScrollToTopButton.vue'
import {getCategoryList} from '@/api/frontend/category'
import {ref} from 'vue'
import {useRouter} from 'vue-router'

const router = useRouter()

// 跳转分类文章列表页
const goCategoryArticleListPage = (id, name) => {
  // 跳转时通过 query 携带参数（分类 ID、分类名称）
  router.push({path: '/category/article/list', query: {id, name}})
}

// 所有分类
const categories = ref([])
getCategoryList({}).then((res) => {
  if (res.success) {
    categories.value = res.data
  }
})
</script>

<style scoped>
/* 可以在这里补充一些特定的过渡效果样式 */
</style>