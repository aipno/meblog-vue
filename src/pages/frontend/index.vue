<template>
  <Header></Header>
  <HeroSection></HeroSection>

  <!-- 主内容区域 -->
  <main class="container max-w-screen-xl mx-auto px-4 md:px-6 py-8 min-h-[60vh]">
    <!-- grid 表格布局，分为 12 列 (PC端) -->
    <div class="grid grid-cols-1 lg:grid-cols-12 gap-8">

      <!-- 左边栏，占用 9 列 -->
      <div class="col-span-1 lg:col-span-9 flex flex-col">

        <!-- 文章列表 -->
        <!-- 优化：xl 屏幕下显示 3 列，卡片更小 -->
        <div class="grid grid-cols-1 md:grid-cols-2 xl:grid-cols-3 gap-6 mb-8">
          <template v-if="articles && articles.length > 0">
            <article v-for="(article, index) in articles" :key="index"
                     :style="{ animationDelay: `${index * 0.1}s` }"
                     class="group bg-white dark:bg-gray-800 border border-gray-100 dark:border-gray-700 rounded-2xl overflow-hidden shadow-sm hover:shadow-lg transition-all duration-300 flex flex-col h-full animate__animated animate__fadeInUp">

              <!-- 封面图 -->
              <!-- 优化：高度减小，更紧凑 -->
              <div class="relative w-full h-40 sm:h-48 overflow-hidden cursor-pointer"
                   @click="goArticleDetailPage(article.id)">
                <img :src="article.cover"
                     alt="cover"
                     class="w-full h-full object-cover transition-transform duration-500 group-hover:scale-110"
                     loading="lazy"/>
                <!-- 遮罩 (仅在hover时显示) -->
                <div
                    class="absolute inset-0 bg-black/10 opacity-0 group-hover:opacity-100 transition-opacity duration-300"></div>

                <!-- 分类标签 (悬浮在左上角) -->
                <span v-if="article.category"
                      class="absolute top-3 left-3 bg-white/90 dark:bg-gray-800/90 backdrop-blur-sm text-[11px] font-bold px-2 py-0.5 rounded-md text-gray-700 dark:text-gray-200 shadow-sm hover:text-sky-600 transition-colors cursor-pointer"
                      @click.stop="goCategoryArticleListPage(article.categoryId, article.category.name)">
                            {{ article.category.name }}
                        </span>
              </div>

              <!-- 内容区 -->
              <!-- 优化：Padding 减小 -->
              <div class="flex-1 p-4 flex flex-col">
                <!-- 标签 -->
                <div v-if="article.tags && article.tags.length > 0" class="flex flex-wrap gap-2 mb-2">
                             <span v-for="(tag, tIndex) in article.tags.slice(0, 3)" :key="tIndex"
                                   class="text-[10px] px-1.5 py-0.5 rounded-full bg-gray-100 dark:bg-gray-700 text-gray-500 dark:text-gray-400 hover:bg-sky-50 hover:text-sky-600 transition-colors cursor-pointer"
                                   @click.stop="goTagArticleListPage(tag.id, tag.name)">
                                 # {{ tag.name }}
                             </span>
                </div>

                <!-- 标题 -->
                <!-- 优化：字号 text-lg -->
                <h2 class="text-lg font-bold text-gray-800 dark:text-white mb-2 line-clamp-2 hover:text-sky-600 dark:hover:text-sky-400 transition-colors cursor-pointer leading-snug"
                    @click="goArticleDetailPage(article.id)">
                  {{ article.title }}
                </h2>

                <!-- 摘要 -->
                <p class="text-xs text-gray-500 dark:text-gray-400 line-clamp-3 mb-3 flex-grow leading-relaxed">
                  {{ article.summary || '暂无摘要' }}
                </p>

                <!-- 底部信息 -->
                <div
                    class="flex items-center justify-between text-xs text-gray-400 border-t border-gray-100 dark:border-gray-700 pt-3 mt-auto">
                  <!-- 发布时间 -->
                  <div class="flex items-center gap-1">
                    <svg class="w-3 h-3" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                      <path d="M8 7V3m8 4V3m-9 8h10M5 21h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v12a2 2 0 002 2z"
                            stroke-linecap="round" stroke-linejoin="round"
                            stroke-width="2"/>
                    </svg>
                    <span>{{ article.createDate }}</span>
                  </div>

                  <span class="group-hover:text-sky-500 transition-colors flex items-center gap-0.5 cursor-pointer"
                        @click="goArticleDetailPage(article.id)">
                                阅读全文 <svg class="w-3 h-3 transition-transform group-hover:translate-x-1" fill="none"
                                              stroke="currentColor" viewBox="0 0 24 24"><path
                      d="M17 8l4 4m0 0l-4 4m4-4H3"
                      stroke-linecap="round"
                      stroke-linejoin="round"
                      stroke-width="2"/></svg>
                             </span>
                </div>
              </div>
            </article>
          </template>

          <!-- 空状态 -->
          <div v-else
               class="col-span-full py-20 text-center bg-white dark:bg-gray-800 rounded-2xl border border-gray-100 dark:border-gray-700">
            <p class="text-gray-400">暂无文章数据</p>
          </div>
        </div>

        <!-- 分页 -->
        <div v-if="pages > 1" class="flex justify-center mt-auto">
          <nav class="inline-flex rounded-md shadow-sm isolate">
            <!-- 上一页 -->
            <button
                :disabled="current <= 1"
                class="relative inline-flex items-center px-4 py-2 text-sm font-medium text-gray-500 bg-white border border-gray-300 rounded-l-md hover:bg-gray-50 focus:z-20 disabled:opacity-50 disabled:cursor-not-allowed dark:bg-gray-800 dark:border-gray-700 dark:text-gray-400 dark:hover:bg-gray-700 transition-colors"
                @click="getArticles(current - 1)">
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
                @click="getArticles(pageNo)">
              {{ index + 1 }}
            </button>

            <!-- 下一页 -->
            <button
                :disabled="current >= pages"
                class="relative inline-flex items-center px-4 py-2 text-sm font-medium text-gray-500 bg-white border border-gray-300 rounded-r-md hover:bg-gray-50 focus:z-20 disabled:opacity-50 disabled:cursor-not-allowed dark:bg-gray-800 dark:border-gray-700 dark:text-gray-400 dark:hover:bg-gray-700 transition-colors"
                @click="getArticles(current + 1)">
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
        <!-- 侧边栏吸附 -->
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
import CategoryListCard from "@/layouts/frontend/components/CategoryListCard.vue";
import TagListCard from "@/layouts/frontend/components/TagListCard.vue";
import HeroSection from "@/layouts/frontend/components/HeroSection.vue";
import ScrollToTopButton from '@/layouts/frontend/components/ScrollToTopButton.vue'
import {initTooltips} from 'flowbite'
import {onMounted, ref} from 'vue'
import {getArticlePageList} from '@/api/frontend/article.js'
import {useRouter} from 'vue-router'

onMounted(() => {
  initTooltips();
})

// 文章集合
const articles = ref([])
// 当前页码
const current = ref(1)
// 每页显示的文章数
const size = ref(12)
// 总文章数
const total = ref(0)
// 总共多少页
const pages = ref(0)

const router = useRouter()

// 跳转文章详情页
const goArticleDetailPage = (articleId) => {
  router.push('/article/' + articleId)
}

// 跳转分类文章列表页
const goCategoryArticleListPage = (id, name) => {
  router.push({path: '/category/article/list', query: {id, name}})
}

// 跳转标签文章列表页
const goTagArticleListPage = (id, name) => {
  router.push({path: '/tag/article/list', query: {id, name}})
}

function getArticles(currentNo) {
  // 上下页是否能点击判断
  if (currentNo < 1 || (pages.value > 0 && currentNo > pages.value)) return

  // 调用分页接口渲染数据
  getArticlePageList({current: currentNo, size: size.value}).then((res) => {
    if (res.success) {
      articles.value = res.data
      current.value = res.current
      size.value = res.size
      total.value = res.total
      pages.value = res.pages

      // 翻页后滚动到顶部（Hero下方）
      if (currentNo !== 1) {
        window.scrollTo({top: window.innerHeight - 80, behavior: 'smooth'})
      }
    }
  })
}

getArticles(current.value)
</script>

<style scoped>
/* 可以在这里补充一些特定的过渡效果样式 */
</style>