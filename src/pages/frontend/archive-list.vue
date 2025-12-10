<template>
  <Header></Header>

  <!-- 主内容区域 -->
  <main class="container max-w-screen-xl mx-auto px-4 md:px-6 py-8 min-h-[80vh]">
    <!-- grid 表格布局，分为 12 列 -->
    <div class="grid grid-cols-12 gap-7">

      <!-- 左边栏，占用 9 列 -->
      <div class="col-span-12 lg:col-span-9 mb-3">

        <!-- 归档列表容器 (时光轴) -->
        <div v-if="archives && archives.length > 0"
             class="relative pl-4 md:pl-8 space-y-10 animate__animated animate__fadeInLeft">
          <!-- 垂直贯穿线 -->
          <div class="absolute left-[7px] md:left-[11px] top-2 bottom-0 w-0.5 bg-gray-100 dark:bg-gray-800"></div>

          <!-- 月份组 -->
          <div v-for="(archive, index) in archives" :key="index" class="relative">
            <!-- 时间轴节点 -->
            <div
                class="absolute -left-[25px] md:-left-[29px] top-1.5 w-4 h-4 md:w-5 md:h-5 rounded-full bg-white dark:bg-gray-900 border-4 border-sky-100 dark:border-sky-900/50 flex items-center justify-center z-10">
              <div class="w-1.5 h-1.5 md:w-2 md:h-2 rounded-full bg-sky-400"></div>
            </div>

            <!-- 月份标题 -->
            <h3 class="text-xl md:text-2xl font-bold text-gray-800 dark:text-gray-100 mb-6 flex items-baseline">
              {{ archive.month }}
              <span class="ml-3 text-sm font-normal text-gray-400">共 {{ archive.articles.length }} 篇</span>
            </h3>

            <!-- 文章列表 -->
            <div class="space-y-4">
              <article v-for="(article, index2) in archive.articles" :key="index2"
                       class="group relative bg-white dark:bg-gray-800 border border-gray-100 dark:border-gray-700 rounded-xl p-4 shadow-sm hover:shadow-md transition-all duration-300 flex items-center gap-4 cursor-pointer"
                       @click="goArticleDetailPage(article.id)">

                <!-- 封面图 -->
                <div
                    class="flex-shrink-0 w-20 h-14 md:w-32 md:h-20 rounded-lg overflow-hidden bg-gray-100 border border-gray-100 dark:border-gray-700/50 relative">
                  <img :src="article.cover"
                       class="w-full h-full object-cover transition-transform duration-500 group-hover:scale-110"
                       loading="lazy"/>
                </div>

                <!-- 内容信息 -->
                <div class="flex-1 min-w-0">
                  <h4 class="text-base md:text-lg font-bold text-gray-700 dark:text-gray-200 mb-1.5 truncate group-hover:text-sky-600 dark:group-hover:text-sky-400 transition-colors">
                    {{ article.title }}
                  </h4>
                  <div class="flex items-center text-xs text-gray-400 gap-4">
                    <!-- 发布日期 -->
                    <span class="flex items-center">
                                    <svg class="w-3.5 h-3.5 mr-1" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path
                                        d="M8 7V3m8 4V3m-9 8h10M5 21h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v12a2 2 0 002 2z"
                                        stroke-linecap="round" stroke-linejoin="round"
                                        stroke-width="2"/></svg>
                                    {{ article.createDate }}
                                </span>
                  </div>
                </div>

                <!-- 悬停箭头 -->
                <div
                    class="hidden md:flex items-center justify-center w-8 h-8 rounded-full bg-gray-50 dark:bg-gray-700 text-gray-300 group-hover:text-sky-500 group-hover:bg-sky-50 dark:group-hover:bg-sky-900/30 transition-all transform translate-x-2 opacity-0 group-hover:opacity-100 group-hover:translate-x-0">
                  <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path d="M9 5l7 7-7 7" stroke-linecap="round" stroke-linejoin="round" stroke-width="2"/>
                  </svg>
                </div>
              </article>
            </div>
          </div>
        </div>

        <!-- 空状态 -->
        <div v-else class="py-20 text-center">
          <p class="text-gray-400">暂无归档数据</p>
        </div>

        <!-- 分页 -->
        <nav v-if="pages > 1" aria-label="Page navigation example" class="mt-10 flex justify-center">
          <ul class="flex items-center -space-x-px h-10 text-base">
            <!-- 上一页 -->
            <li>
              <a :class="[current > 1 ? 'hover:bg-gray-100 hover:text-gray-700 dark:hover:bg-gray-700 dark:hover:text-white' : 'cursor-not-allowed opacity-50']"
                 class="flex items-center justify-center px-4 h-10 ml-0 leading-tight text-gray-500 bg-white border border-gray-300 rounded-l-lg dark:bg-gray-800 dark:border-gray-700 dark:text-gray-400"
                 @click="getArchives(current - 1)">
                <span class="sr-only">上一页</span>
                <svg aria-hidden="true" class="w-3 h-3" fill="none" viewBox="0 0 6 10"
                     xmlns="http://www.w3.org/2000/svg">
                  <path d="M5 1 1 5l4 4" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round"
                        stroke-width="2"/>
                </svg>
              </a>
            </li>
            <!-- 页码 -->
            <li v-for="(pageNo, index) in pages" :key="index">
              <a :class="[pageNo === current ? 'text-sky-600 bg-sky-50 border-sky-500 hover:bg-sky-100 hover:text-sky-700 dark:bg-gray-700 dark:text-white' : 'text-gray-500 border-gray-300 bg-white hover:bg-gray-100 hover:text-gray-700 dark:bg-gray-800 dark:border-gray-700 dark:text-gray-400 dark:hover:bg-gray-700 dark:hover:text-white']"
                 class="flex items-center justify-center px-4 h-10 leading-tight border transition-colors cursor-pointer"
                 @click="getArchives(pageNo)">
                {{ index + 1 }}
              </a>
            </li>
            <!-- 下一页 -->
            <li>
              <a :class="[current < pages ? 'hover:bg-gray-100 hover:text-gray-700 dark:hover:bg-gray-700 dark:hover:text-white' : 'cursor-not-allowed opacity-50']"
                 class="flex items-center justify-center px-4 h-10 leading-tight text-gray-500 bg-white border border-gray-300 rounded-r-lg dark:bg-gray-800 dark:border-gray-700 dark:text-gray-400"
                 @click="getArchives(current + 1)">
                <span class="sr-only">下一页</span>
                <svg aria-hidden="true" class="w-3 h-3" fill="none" viewBox="0 0 6 10"
                     xmlns="http://www.w3.org/2000/svg">
                  <path d="m1 9 4-4-4-4" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round"
                        stroke-width="2"/>
                </svg>
              </a>
            </li>
          </ul>
        </nav>

      </div>

      <!-- 右边侧边栏，占用 3 列 -->
      <aside class="col-span-12 lg:col-span-3">
        <div class="sticky top-[5.5rem]">
          <!-- 博主信息 -->
          <UserInfoCard></UserInfoCard>

          <!-- 分类 -->
          <CategoryListCard></CategoryListCard>

          <!-- 标签 -->
          <TagListCard></TagListCard>
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
import {getArchivePageList} from '@/api/frontend/archive.js'
import {ref} from 'vue'
import {useRouter} from 'vue-router'

// 文章归档
const archives = ref([])
// 当前页码
const current = ref(1)
// 每页显示的文章数
const size = ref(10)
// 总文章数
const total = ref(0)
// 总共多少页
const pages = ref(0)

const router = useRouter()

// 跳转文章详情页
const goArticleDetailPage = (articleId) => {
  router.push('/article/' + articleId)
}

function getArchives(currentNo) {
  // 上下页是否能点击判断
  if (currentNo < 1 || (pages.value > 0 && currentNo > pages.value)) return

  // 调用分页接口渲染数据
  getArchivePageList({current: currentNo, size: size.value}).then((res) => {
    if (res.success) {
      archives.value = res.data
      current.value = res.current
      size.value = res.size
      total.value = res.total
      pages.value = res.pages
    }
  })
}

getArchives(current.value)
</script>