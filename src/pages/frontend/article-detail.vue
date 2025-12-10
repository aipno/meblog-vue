<template>
  <Header></Header>

  <!-- 顶部 Hero Banner 区域 -->
  <!-- 使用 v-if="success" 确保数据加载后再显示，避免闪烁 -->
  <div v-if="success" class="relative w-full h-[400px] md:h-[500px] flex items-end animate__animated animate__fadeIn">
    <!-- 背景图层 -->
    <div class="absolute inset-0 z-0 overflow-hidden">
      <img v-if="article.cover" :src="article.cover"
           alt="cover"
           class="w-full h-full object-cover transition-transform duration-700 hover:scale-105 filter brightness-75"/>
      <div v-else class="w-full h-full bg-gradient-to-br from-slate-800 to-slate-900"></div>
      <!-- 渐变遮罩，保证文字清晰度 -->
      <div class="absolute inset-0 bg-gradient-to-t from-gray-900 via-gray-900/40 to-transparent"></div>
    </div>

    <!-- 文章信息容器 -->
    <div class="relative z-10 container max-w-screen-xl mx-auto px-6 pb-12 md:pb-16 text-white">
      <!-- 标签 -->
      <div class="flex flex-wrap gap-2 mb-4 animate__animated animate__fadeInUp">
            <span v-for="(tag, index) in article.tags" :key="index"
                  class="px-3 py-1 text-xs font-medium bg-white/20 backdrop-blur-md border border-white/20 rounded-full hover:bg-white/30 transition-colors cursor-pointer"
                  @click.stop="goTagArticleListPage(tag.id, tag.name)">
              # {{ tag.name }}
            </span>
      </div>

      <!-- 标题 -->
      <h1 class="text-3xl md:text-5xl font-extrabold mb-6 leading-tight tracking-tight text-white drop-shadow-md animate__animated animate__fadeInUp animate__delay-200ms">
        {{ article.title }}
      </h1>

      <!-- Meta 信息 -->
      <div
          class="flex flex-wrap items-center gap-6 text-sm text-gray-300 animate__animated animate__fadeInUp animate__delay-300ms font-medium">
        <!-- 分类 -->
        <div class="flex items-center gap-1.5 hover:text-white transition-colors cursor-pointer"
             @click.stop="goCategoryArticleListPage(article.categoryId, article.categoryName)">
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path d="M3 7v10a2 2 0 002 2h14a2 2 0 002-2V9a2 2 0 00-2-2h-6l-2-2H5a2 2 0 00-2 2z" stroke-linecap="round"
                  stroke-linejoin="round"
                  stroke-width="2"/>
          </svg>
          {{ article.categoryName }}
        </div>

        <!-- 发布时间 -->
        <div class="flex items-center gap-1.5">
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path d="M8 7V3m8 4V3m-9 8h10M5 21h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v12a2 2 0 002 2z"
                  stroke-linecap="round" stroke-linejoin="round"
                  stroke-width="2"/>
          </svg>
          {{ article.createTime }}
        </div>

        <!-- 字数 -->
        <div class="flex items-center gap-1.5 hidden sm:flex">
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path
                d="M9 12h6m-6 4h6m2 5H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z"
                stroke-linecap="round" stroke-linejoin="round"
                stroke-width="2"/>
          </svg>
          {{ article.totalWords }} 字
        </div>

        <!-- 阅读时长 -->
        <div class="flex items-center gap-1.5 hidden sm:flex">
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path d="M12 8v4l3 3m6-3a9 9 0 11-18 0 9 9 0 0118 0z" stroke-linecap="round" stroke-linejoin="round"
                  stroke-width="2"/>
          </svg>
          {{ article.readTime }}
        </div>

        <!-- 阅读量 -->
        <div class="flex items-center gap-1.5">
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path d="M15 12a3 3 0 11-6 0 3 3 0 016 0z" stroke-linecap="round" stroke-linejoin="round" stroke-width="2"/>
            <path
                d="M2.458 12C3.732 7.943 7.523 5 12 5c4.478 0 8.268 2.943 9.542 7-1.274 4.057-5.064 7-9.542 7-4.477 0-8.268-2.943-9.542-7z"
                stroke-linecap="round" stroke-linejoin="round"
                stroke-width="2"/>
          </svg>
          {{ article.readNum }}
        </div>
      </div>
    </div>
  </div>


  <!-- 主内容区域 -->
  <main class="container max-w-screen-xl mx-auto px-4 md:px-6 py-10">
    <div class="grid grid-cols-12 gap-8">

      <!-- 左侧内容区 -->
      <div class="col-span-12 lg:col-span-9">
        <div
            class="bg-white border border-gray-100 rounded-2xl shadow-sm dark:bg-gray-800 dark:border-gray-700 overflow-hidden min-h-[50vh] transition-colors duration-300">

          <!-- 正文内容 -->
          <div class="p-6 md:p-10 lg:p-12">
            <div :class="{ 'dark': isDark }">
              <div ref="articleContentRef" v-viewer class="article-content" v-html="article.content"></div>
            </div>

            <!-- 权限提示 -->
            <NoPermission v-if="articlePermission" message="该文章仅限登录用户查看，请登录后继续阅读。"></NoPermission>
          </div>

          <!-- 底部信息 -->
          <div v-if="success"
               class="px-6 md:px-12 py-6 border-t border-gray-100 dark:border-gray-700 bg-gray-50/50 dark:bg-gray-800/50">
            <div
                class="flex flex-col md:flex-row items-center justify-between text-sm text-gray-500 dark:text-gray-400 gap-4">
              <div class="flex items-center gap-2">
                <span class="font-medium">最后编辑于：</span>
                <span>{{ article.updateTime }}</span>
              </div>
              <!-- 可以在这里加分享按钮等 -->
            </div>
          </div>
        </div>

        <!-- 上下篇导航 (预留样式，逻辑接入请参考Wiki) -->
        <nav v-if="preNext" class="grid grid-cols-1 md:grid-cols-2 gap-4 mt-8">
          <div v-if="preNext.preArticle"
               class="group p-5 bg-white border border-gray-100 rounded-xl cursor-pointer hover:shadow-md hover:border-sky-200 dark:bg-gray-800 dark:border-gray-700 dark:hover:border-gray-600 transition-all"
               @click="goArticleDetailPage(preNext.preArticle.articleId)">
            <div class="text-xs text-gray-400 mb-1 group-hover:text-sky-600 transition-colors">上一篇</div>
            <div class="text-gray-700 dark:text-gray-200 font-medium truncate">{{
                preNext.preArticle.articleTitle
              }}
            </div>
          </div>
          <div v-else class="hidden md:block"></div> <!-- 占位 -->

          <div v-if="preNext.nextArticle"
               class="group p-5 bg-white border border-gray-100 rounded-xl cursor-pointer hover:shadow-md hover:border-sky-200 dark:bg-gray-800 dark:border-gray-700 dark:hover:border-gray-600 transition-all text-right"
               @click="goArticleDetailPage(preNext.nextArticle.articleId)">
            <div class="text-xs text-gray-400 mb-1 group-hover:text-sky-600 transition-colors">下一篇</div>
            <div class="text-gray-700 dark:text-gray-200 font-medium truncate">{{
                preNext.nextArticle.articleTitle
              }}
            </div>
          </div>
        </nav>
      </div>

      <!-- 右侧侧边栏 -->
      <aside class="col-span-12 lg:col-span-3 space-y-6">
        <!-- 侧边栏吸附 -->
        <div class="sticky top-[5.5rem] space-y-6 transition-all duration-300">
          <!-- 博主信息 -->
          <UserInfoCard class="animate__animated animate__fadeInRight"/>

          <!-- 目录 (仅在有目录数据时显示) -->
          <Toc class="animate__animated animate__fadeInRight animate__delay-100ms"/>

          <!-- 分类 -->
          <CategoryListCard class="animate__animated animate__fadeInRight animate__delay-200ms"/>

          <!-- 标签 -->
          <TagListCard class="animate__animated animate__fadeInRight animate__delay-300ms"/>
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
import NoPermission from "@/layouts/frontend/components/NoPermission.vue";
import {getArticleDetail} from '@/api/frontend/article'
import {useRoute, useRouter} from 'vue-router'
import {nextTick, onMounted, ref, watch} from 'vue'
import Toc from '@/layouts/frontend/components/Toc.vue'
import hljs from 'highlight.js'
import 'highlight.js/styles/tokyo-night-dark.css' // 保持代码高亮样式
import ScrollToTopButton from "@/layouts/frontend/components/ScrollToTopButton.vue";
import {initTooltips} from 'flowbite'
import {useDark} from '@vueuse/core'

const isDark = useDark()

onMounted(() => {
  initTooltips();
})

const route = useRoute()
const router = useRouter()

const article = ref({})
const success = ref(false)
const articlePermission = ref(false)
const preNext = ref(null) // 预留上下篇数据

// 获取文章详情
function refreshArticleDetail(articleId) {
  success.value = false // 重置状态，触发 loading 或骨架屏（如果有）
  articlePermission.value = false

  getArticleDetail(articleId).then((res) => {
    if (!res.success) {
      if (res.errorCode === '20002') {
        articlePermission.value = true
        return
      }
      if (res.errorCode === '20010') {
        router.push({name: 'NotFound'})
        return
      }
      return
    }

    article.value = res.data
    success.value = true

    // 处理代码高亮和复制按钮
    nextTick(() => {
      // 代码高亮
      document.querySelectorAll('pre code').forEach((block) => {
        hljs.highlightElement(block)
      })

      // 添加复制按钮
      document.querySelectorAll('pre').forEach(preElement => {
        // 避免重复添加
        if (preElement.querySelector('.copy-code-btn')) return;

        let firstCode = preElement.querySelector('code');
        if (firstCode) {
          // 使用更现代的图标按钮 HTML
          let copyCodeBtn = document.createElement('button');
          copyCodeBtn.className = 'copy-code-btn absolute top-2 right-2 p-1.5 rounded-md bg-gray-700/50 hover:bg-gray-600 text-gray-300 hover:text-white transition-all opacity-0 group-hover:opacity-100 focus:outline-none';
          copyCodeBtn.innerHTML = `
            <svg class="w-4 h-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
               <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 16H6a2 2 0 01-2-2V6a2 2 0 012-2h8a2 2 0 012 2v2m-6 12h8a2 2 0 002-2v-8a2 2 0 00-2-2h-8a2 2 0 00-2 2v8a2 2 0 002 2z" />
            </svg>
          `;

          // 给 pre 添加 group 类以支持 hover 显示
          preElement.classList.add('group', 'relative');
          preElement.appendChild(copyCodeBtn);

          copyCodeBtn.addEventListener('click', () => {
            copyToClipboard(preElement.textContent);

            // 复制成功反馈
            const originalIcon = copyCodeBtn.innerHTML;
            copyCodeBtn.innerHTML = `<svg class="w-4 h-4 text-green-400" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7" /></svg>`;
            setTimeout(() => {
              copyCodeBtn.innerHTML = originalIcon;
            }, 2000);
          });
        }
      })
    })
  })
}

function copyToClipboard(text) {
  // 现代浏览器复制 API
  if (navigator.clipboard && window.isSecureContext) {
    navigator.clipboard.writeText(text);
  } else {
    // 兼容性回退
    const textarea = document.createElement('textarea');
    textarea.value = text;
    textarea.style.position = 'absolute';
    textarea.style.left = '-9999px';
    document.body.appendChild(textarea);
    textarea.select();
    document.execCommand('copy');
    document.body.removeChild(textarea);
  }
}

refreshArticleDetail(route.params.articleId)

// 路由跳转辅助
const goCategoryArticleListPage = (id, name) => {
  router.push({path: '/category/article/list', query: {id, name}})
}
const goTagArticleListPage = (id, name) => {
  router.push({path: '/tag/article/list', query: {id, name}})
}

watch(route, (newRoute) => {
  refreshArticleDetail(newRoute.params.articleId)
})
</script>

<style scoped>
/* 文章内容排版优化 (Tailwind Typography 风格复刻) */
::v-deep(.article-content) {
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, "Segoe UI", Helvetica, Arial, sans-serif, "Apple Color Emoji", "Segoe UI Emoji";
  line-height: 1.8;
  color: #374151; /* gray-700 */
  font-size: 1.05rem;
}

/* 标题 */
::v-deep(.article-content h1),
::v-deep(.article-content h2),
::v-deep(.article-content h3) {
  color: #111827; /* gray-900 */
  font-weight: 700;
  margin-top: 2.5em;
  margin-bottom: 1em;
  line-height: 1.3;
}

::v-deep(.article-content h1) {
  font-size: 2.25em;
  border-bottom: 1px solid #e5e7eb;
  padding-bottom: 0.3em;
}

::v-deep(.article-content h2) {
  font-size: 1.75em;
  padding-left: 12px;
  border-left: 4px solid #0284c7;
}

/* Sky-600 */
::v-deep(.article-content h3) {
  font-size: 1.4em;
}

/* 段落 */
::v-deep(.article-content p) {
  margin-top: 1.25em;
  margin-bottom: 1.25em;
  text-align: justify;
}

/* 列表 */
::v-deep(.article-content ul),
::v-deep(.article-content ol) {
  margin-top: 1.25em;
  margin-bottom: 1.25em;
  padding-left: 1.625em;
}

::v-deep(.article-content ul) {
  list-style-type: disc;
}

::v-deep(.article-content ol) {
  list-style-type: decimal;
}

::v-deep(.article-content li) {
  margin-top: 0.5em;
  margin-bottom: 0.5em;
}

/* 引用 */
::v-deep(.article-content blockquote) {
  font-style: italic;
  color: #4b5563; /* gray-600 */
  border-left-width: 0.25rem;
  border-left-color: #e5e7eb; /* gray-200 */
  margin-top: 1.6em;
  margin-bottom: 1.6em;
  padding-left: 1em;
  background: #f9fafb; /* gray-50 */
  padding: 1rem;
  border-radius: 0.5rem;
}

/* 代码块 */
::v-deep(.article-content pre) {
  background-color: #1e293b !important; /* slate-800 */
  color: #e2e8f0;
  border-radius: 0.75rem;
  overflow-x: auto;
  margin-top: 1.71em;
  margin-bottom: 1.71em;
  padding: 1.25em;
  font-size: 0.9em;
  box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
}

::v-deep(.article-content code) {
  color: #ef4444; /* red-500 */
  background-color: #fee2e2; /* red-100 */
  padding: 0.2em 0.4em;
  border-radius: 0.25rem;
  font-size: 0.875em;
  font-weight: 600;
  font-family: 'Menlo', 'Monaco', 'Courier New', monospace;
}

::v-deep(.article-content pre code) {
  color: inherit;
  background-color: transparent;
  padding: 0;
  font-weight: 400;
  border-radius: 0;
}

/* 图片 */
::v-deep(.article-content img) {
  margin-top: 2em;
  margin-bottom: 2em;
  border-radius: 0.75rem;
  box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
  max-width: 100%;
  height: auto;
  display: block;
  margin-left: auto;
  margin-right: auto;
}

/* 链接 */
::v-deep(.article-content a) {
  color: #0284c7; /* sky-600 */
  text-decoration: none;
  border-bottom: 1px solid transparent;
  transition: border-color 0.2s;
}

::v-deep(.article-content a:hover) {
  border-bottom-color: #0284c7;
}

/* 水平线 */
::v-deep(.article-content hr) {
  margin-top: 3em;
  margin-bottom: 3em;
  border-top-width: 1px;
  border-color: #e5e7eb;
}

/* 表格 */
::v-deep(.article-content table) {
  width: 100%;
  table-layout: auto;
  text-align: left;
  margin-top: 2em;
  margin-bottom: 2em;
  font-size: 0.875em;
  line-height: 1.7142857;
  border-collapse: collapse;
}

::v-deep(.article-content thead) {
  border-bottom-width: 1px;
  border-bottom-color: #d1d5db; /* gray-300 */
}

::v-deep(.article-content thead th) {
  color: #111827;
  font-weight: 600;
  vertical-align: bottom;
  padding-right: 0.5714286em;
  padding-bottom: 0.5714286em;
  padding-left: 0.5714286em;
}

::v-deep(.article-content tbody tr) {
  border-bottom-width: 1px;
  border-bottom-color: #e5e7eb;
}

::v-deep(.article-content tbody tr:last-child) {
  border-bottom-width: 0;
}

::v-deep(.article-content tbody td) {
  vertical-align: baseline;
  padding-top: 0.5714286em;
  padding-right: 0.5714286em;
  padding-bottom: 0.5714286em;
  padding-left: 0.5714286em;
}

/* === 暗黑模式适配 === */
::v-deep(.dark .article-content) {
  color: #d1d5db; /* gray-300 */
}

::v-deep(.dark .article-content h1),
::v-deep(.dark .article-content h2),
::v-deep(.dark .article-content h3) {
  color: #f3f4f6; /* gray-100 */
}

::v-deep(.dark .article-content h1) {
  border-bottom-color: #374151;
}

::v-deep(.dark .article-content blockquote) {
  color: #9ca3af;
  border-left-color: #4b5563;
  background: #1f2937;
}

::v-deep(.dark .article-content code) {
  color: #fca5a5;
  background-color: #7f1d1d;
}

/* red-300, red-900 */
::v-deep(.dark .article-content pre code) {
  color: inherit;
  background-color: transparent;
}

::v-deep(.dark .article-content a) {
  color: #38bdf8;
}

/* sky-400 */
::v-deep(.dark .article-content a:hover) {
  border-bottom-color: #38bdf8;
}

::v-deep(.dark .article-content hr) {
  border-color: #374151;
}

::v-deep(.dark .article-content thead) {
  border-bottom-color: #4b5563;
}

::v-deep(.dark .article-content thead th) {
  color: #f3f4f6;
}

::v-deep(.dark .article-content tbody tr) {
  border-bottom-color: #374151;
}
</style>