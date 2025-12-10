<template>
  <Header></Header>

  <!-- 主内容区域 -->
  <main class="container max-w-screen-xl mx-auto px-4 md:px-6 py-8 min-h-[80vh]">
    <!-- grid 表格布局，分为 12 列 (PC端) -->
    <div class="grid grid-cols-1 lg:grid-cols-12 gap-8">

      <!-- 左边栏，占用 9 列 -->
      <div class="col-span-1 lg:col-span-9 space-y-6">

        <!-- 顶部：分类筛选卡片 -->
        <div
            class="w-full p-6 bg-white border border-gray-100 rounded-2xl shadow-sm dark:bg-gray-800 dark:border-gray-700 transition-all animate__animated animate__fadeInDown">
          <!-- 标题栏 -->
          <div class="flex items-center justify-between mb-4 pb-4 border-b border-gray-100 dark:border-gray-700">
            <h2 class="flex items-center text-lg font-bold text-gray-800 dark:text-gray-100">
              <span class="p-2 mr-3 rounded-lg bg-indigo-50 text-indigo-600 dark:bg-indigo-900/30 dark:text-indigo-400">
                 <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path d="M19 11H5m14 0a2 2 0 012 2v6a2 2 0 01-2 2H5a2 2 0 01-2-2v-6a2 2 0 012-2m14 0V9a2 2 0 00-2-2M5 11V9a2 2 0 012-2m0 0V5a2 2 0 012-2h6a2 2 0 012 2v2M7 7h10"
                                                                                                  stroke-linecap="round"
                                                                                                  stroke-linejoin="round"
                                                                                                  stroke-width="2"/></svg>
              </span>
              分类检索
              <span v-if="route.query.name" class="ml-2 text-sm font-normal text-gray-500 dark:text-gray-400">
                 - <span class="font-bold text-sky-600 dark:text-sky-400">{{ route.query.name }}</span>
              </span>
            </h2>
            <span class="text-xs text-gray-400 bg-gray-50 px-2 py-1 rounded-md dark:bg-gray-700">
               共 {{ categories.length }} 个分类
            </span>
          </div>

          <!-- 分类列表 (胶囊样式) -->
          <div class="flex flex-wrap gap-3">
            <a v-for="(category, index) in categories" :key="index"
               :class="[
                   route.query.id == category.id
                   ? 'bg-sky-500 text-white border-sky-500 shadow-md shadow-sky-200 dark:shadow-none'
                   : 'bg-white text-gray-600 border-gray-200 hover:border-sky-300 hover:text-sky-600 hover:bg-sky-50 dark:bg-gray-700 dark:border-gray-600 dark:text-gray-300 dark:hover:bg-gray-600 dark:hover:text-white'
               ]"
               class="group relative inline-flex items-center px-4 py-2 text-sm font-medium rounded-full cursor-pointer transition-all duration-200 border select-none"
               @click="goCategoryArticleListPage(category.id, category.name)">
              <span>{{ category.name }}</span>
              <span :class="[route.query.id == category.id ? 'text-white' : 'text-gray-400 group-hover:text-sky-600 dark:text-gray-400 dark:group-hover:text-white']"
                    class="ml-2 text-xs opacity-80 bg-black/10 dark:bg-white/20 px-1.5 py-0.5 rounded-full">
                  {{ category.articlesTotal }}
              </span>
            </a>
          </div>
        </div>

        <!-- 文章列表 -->
        <div class="space-y-5 animate__animated animate__fadeInUp">
          <template v-if="articles && articles.length > 0">
            <article v-for="(article, index) in articles" :key="index"
                     class="group relative flex flex-col md:flex-row bg-white dark:bg-gray-800 border border-gray-100 dark:border-gray-700 rounded-2xl overflow-hidden shadow-sm hover:shadow-lg transition-all duration-300 cursor-pointer"
                     @click="goArticleDetailPage(article.id)">

              <!-- 封面图 -->
              <div class="w-full md:w-64 h-48 md:h-auto flex-shrink-0 overflow-hidden relative">
                <img :src="article.cover"
                     class="w-full h-full object-cover transition-transform duration-500 group-hover:scale-110"
                     loading="lazy"/>
                <!-- 遮罩 (仅在hover时显示) -->
                <div
                    class="absolute inset-0 bg-black/20 opacity-0 group-hover:opacity-100 transition-opacity duration-300"></div>
              </div>

              <!-- 内容区 -->
              <div class="flex-1 p-5 md:p-6 flex flex-col justify-center">
                <div class="flex items-center gap-2 text-xs text-gray-400 mb-3">
                         <span
                             class="flex items-center bg-gray-100 dark:bg-gray-700 px-2 py-0.5 rounded text-gray-500 dark:text-gray-300">
                             <svg class="w-3 h-3 mr-1" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path
                                 d="M8 7V3m8 4V3m-9 8h10M5 21h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v12a2 2 0 002 2z" stroke-linecap="round" stroke-linejoin="round"
                                 stroke-width="2"/></svg>
                             {{ article.createDate }}
                         </span>
                  <!-- 这里的标签可以从 article 数据中获取，如果接口有返回 -->
                </div>

                <h3 class="text-xl font-bold text-gray-800 dark:text-white mb-2 line-clamp-2 group-hover:text-sky-600 dark:group-hover:text-sky-400 transition-colors">
                  {{ article.title }}
                </h3>

                <p class="text-sm text-gray-500 dark:text-gray-400 line-clamp-2 mb-4">
                  {{ article.summary || '暂无摘要' }}
                </p>

                <div class="mt-auto flex items-center justify-between">
                         <span
                             class="text-sky-500 text-sm font-medium flex items-center opacity-0 -translate-x-2 group-hover:opacity-100 group-hover:translate-x-0 transition-all duration-300">
                             阅读全文 <svg class="w-4 h-4 ml-1" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path
                             d="M17 8l4 4m0 0l-4 4m4-4H3" stroke-linecap="round" stroke-linejoin="round"
                             stroke-width="2"/></svg>
                         </span>
                </div>
              </div>
            </article>
          </template>

          <!-- 空状态 -->
          <div v-else
               class="w-full py-16 bg-white dark:bg-gray-800 border border-gray-100 dark:border-gray-700 rounded-2xl flex flex-col items-center justify-center text-center">
            <!-- 复用 SVG 插画，但限制大小 -->
            <div class="w-64 h-64 mb-6">
              <!-- 此处插入原 SVG 代码，为节省篇幅省略，请保留原文件中的 SVG -->
              <svg class="w-full h-full opacity-80" viewBox="0 0 600 600" xmlns="http://www.w3.org/2000/svg">
                <defs>
                  <!-- 定义一些简单的渐变以适配空状态 -->
                  <linearGradient id="empty-grad" x1="0%" x2="100%" y1="0%" y2="100%">
                    <stop offset="0%" style="stop-color:#f3f4f6;stop-opacity:1"/>
                    <stop offset="100%" style="stop-color:#e5e7eb;stop-opacity:1"/>
                  </linearGradient>
                </defs>
                <!-- 简化的空状态占位图，实际请使用原文件中的 SVG -->
                <circle cx="300" cy="300" fill="url(#empty-grad)" r="200"/>
                <text dominant-baseline="middle" fill="#9ca3af" font-family="sans-serif" font-size="24" text-anchor="middle"
                      x="50%" y="50%">暂无文章
                </text>
              </svg>
            </div>
            <h3 class="text-lg font-medium text-gray-900 dark:text-white">暂无相关文章</h3>
            <p class="text-gray-500 dark:text-gray-400 mt-2 text-sm">该分类下还没有发布任何文章，去看看其他分类吧~</p>
          </div>
        </div>

        <!-- 分页 -->
        <div v-if="total > 0" class="flex justify-center mt-10">
          <nav class="inline-flex rounded-md shadow-sm isolate">
            <!-- 上一页 -->
            <button
                :disabled="current <= 1"
                class="relative inline-flex items-center px-4 py-2 text-sm font-medium text-gray-500 bg-white border border-gray-300 rounded-l-md hover:bg-gray-50 focus:z-20 disabled:opacity-50 disabled:cursor-not-allowed dark:bg-gray-800 dark:border-gray-700 dark:text-gray-400 dark:hover:bg-gray-700 transition-colors"
                @click="getCategoryArticles(current - 1)">
              <span class="sr-only">Previous</span>
              <svg aria-hidden="true" class="w-5 h-5" fill="currentColor" viewBox="0 0 20 20">
                <path clip-rule="evenodd"
                      d="M12.79 5.23a.75.75 0 01-.02 1.06L8.832 10l3.938 3.71a.75.75 0 11-1.04 1.08l-4.5-4.25a.75.75 0 010-1.08l4.5-4.25a.75.75 0 011.06.02z"
                      fill-rule="evenodd"/>
              </svg>
            </button>

            <!-- 页码 -->
            <button
                v-for="(pageNo, index) in pages" :key="index"
                :class="[
                    pageNo === current
                    ? 'z-10 bg-sky-50 border-sky-500 text-sky-600 dark:bg-sky-900/30 dark:border-sky-500 dark:text-sky-400'
                    : 'bg-white border-gray-300 text-gray-500 hover:bg-gray-50 dark:bg-gray-800 dark:border-gray-700 dark:text-gray-400 dark:hover:bg-gray-700'
                ]"
                class="relative inline-flex items-center px-4 py-2 text-sm font-medium border focus:z-20 transition-colors"
                @click="getCategoryArticles(pageNo)">
              {{ index + 1 }}
            </button>

            <!-- 下一页 -->
            <button
                :disabled="current >= pages"
                class="relative inline-flex items-center px-4 py-2 text-sm font-medium text-gray-500 bg-white border border-gray-300 rounded-r-md hover:bg-gray-50 focus:z-20 disabled:opacity-50 disabled:cursor-not-allowed dark:bg-gray-800 dark:border-gray-700 dark:text-gray-400 dark:hover:bg-gray-700 transition-colors"
                @click="getCategoryArticles(current + 1)">
              <span class="sr-only">Next</span>
              <svg aria-hidden="true" class="w-5 h-5" fill="currentColor" viewBox="0 0 20 20">
                <path clip-rule="evenodd"
                      d="M7.21 14.77a.75.75 0 01.02-1.06L11.168 10 7.23 6.29a.75.75 0 111.04-1.08l4.5 4.25a.75.75 0 010 1.08l-4.5 4.25a.75.75 0 01-1.06-.02z"
                      fill-rule="evenodd"/>
              </svg>
            </button>
          </nav>
        </div>

      </div>

      <!-- 右边侧边栏，占用 3 列 -->
      <aside class="col-span-1 lg:col-span-3 space-y-6">
        <div class="sticky top-[5.5rem] space-y-6">
          <!-- 博主信息 -->
          <UserInfoCard class="animate__animated animate__fadeInRight"/>

          <!-- 分类 -->
          <CategoryListCard class="animate__animated animate__fadeInRight animate__delay-100ms"/>

          <!-- 标签 -->
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
import CategoryListCard from '@/layouts/frontend/components/CategoryListCard.vue'
import ScrollToTopButton from '@/layouts/frontend/components/ScrollToTopButton.vue'
import {ref, watch} from 'vue'
import {useRoute, useRouter} from 'vue-router'
import {getCategoryArticlePageList, getCategoryList} from '@/api/frontend/category'

const route = useRoute()
const router = useRouter()

// 文章集合
const articles = ref([])
// 分类名称
const categoryName = ref(route.query.name)
// 分类 ID
const categoryId = ref(route.query.id)

// 监听路由变化，重新获取数据
watch(route, (newRoute) => {
  categoryName.value = newRoute.query.name
  categoryId.value = newRoute.query.id
  getCategoryArticles(1) // 切换分类时重置为第一页
})

// 当前页码
const current = ref(1)
// 每页显示的文章数
const size = ref(10)
// 总文章数
const total = ref(0)
// 总共多少页
const pages = ref(0)

function getCategoryArticles(currentNo) {
  // 边界检查
  if (currentNo < 1 || (pages.value > 0 && currentNo > pages.value)) return

  // 调用分页接口
  getCategoryArticlePageList({current: currentNo, size: size.value, id: categoryId.value}).then((res) => {
    if (res.success) {
      articles.value = res.data
      current.value = res.current
      size.value = res.size
      total.value = res.total
      pages.value = res.pages

      // 滚动到顶部，提升体验
      window.scrollTo({top: 0, behavior: 'smooth'})
    }
  })
}

// 初始化加载
if (categoryId.value) {
  getCategoryArticles(current.value)
}

// 跳转文章详情页
const goArticleDetailPage = (articleId) => {
  router.push('/article/' + articleId)
}

// 所有分类
const categories = ref([])
getCategoryList({}).then((res) => {
  if (res.success) {
    categories.value = res.data
  }
})

// 跳转分类文章列表页 (点击顶部筛选卡片)
const goCategoryArticleListPage = (id, name) => {
  router.push({path: '/category/article/list', query: {id, name}})
}
</script>

<style scoped>
/* 可以在这里补充一些特定的过渡效果样式，如果 Tailwind 不能完全满足 */
</style>