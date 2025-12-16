<template>
  <div class="main min-h-screen flex flex-col bg-gray-50 dark:bg-[#0d1117] transition-colors duration-300 font-sans">
    <WikiHeader></WikiHeader>

    <main class="grow container max-w-screen-3xl mx-auto px-4 sm:px-6 md:px-8 py-6 relative flex items-start gap-6">

      <!-- 移动端目录遮罩 -->
      <div v-if="showMobileCatalog" class="fixed inset-0 z-40 bg-black/50 lg:hidden backdrop-blur-sm transition-opacity"
        @click="showMobileCatalog = false"></div>

      <!-- 左侧：知识库目录 (Sidebar) -->
      <aside :class="[
        isExpand ? 'lg:w-72 lg:opacity-100 lg:translate-x-0' : 'lg:w-0 lg:opacity-0 lg:-translate-x-4',
        showMobileCatalog ? 'translate-x-0' : '-translate-x-full lg:translate-x-0'
      ]"
        class="fixed inset-y-0 left-0 z-50 w-72 bg-white dark:bg-[#0d1117] border-r border-gray-200 dark:border-gray-800 transform transition-all duration-300 ease-in-out lg:static lg:z-0 lg:h-[calc(100vh-5rem)] lg:sticky lg:top-20 overflow-y-auto custom-scrollbar shadow-2xl lg:shadow-none">

        <div
          class="p-4 sticky top-0 bg-white dark:bg-[#0d1117] z-10 flex items-center justify-between border-b border-gray-100 dark:border-gray-800 mb-2">
          <h3 class="text-sm font-bold text-gray-900 dark:text-gray-100 flex items-center gap-2">
            <svg class="w-4 h-4 text-sky-500" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                d="M19 11H5m14 0a2 2 0 012 2v6a2 2 0 01-2 2H5a2 2 0 01-2-2v-6a2 2 0 012-2m14 0V9a2 2 0 00-2-2M5 11V9a2 2 0 012-2m0 0V5a2 2 0 012-2h6a2 2 0 012 2v2M7 7h10" />
            </svg>
            目录结构
          </h3>
          <button @click="showMobileCatalog = false"
            class="lg:hidden p-1 text-gray-500 hover:bg-gray-100 dark:hover:bg-gray-800 rounded-md">
            <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
            </svg>
          </button>
        </div>

        <!-- 目录列表 -->
        <div class="px-3 pb-8 space-y-1">
          <div v-for="(catalog, index) in catalogs" :key="index" class="space-y-1">
            <!-- 一级目录 -->
            <div
              class="group flex items-center justify-between px-3 py-2.5 text-sm font-medium text-gray-700 dark:text-gray-200 rounded-lg hover:bg-gray-100 dark:hover:bg-gray-800/50 cursor-pointer transition-colors select-none"
              @click="toggleCatalog(catalog.id)">
              <span class="truncate" v-html="catalog.title"></span>
              <svg :class="{ 'rotate-90': !collapsedCatalogs.includes(catalog.id) }"
                class="w-4 h-4 text-gray-400 transition-transform duration-200" fill="none" stroke="currentColor"
                viewBox="0 0 24 24">
                <path d="M9 5l7 7-7 7" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" />
              </svg>
            </div>

            <!-- 二级目录 (展开/收起) -->
            <ul v-show="!collapsedCatalogs.includes(catalog.id)" class="pl-4 space-y-0.5 relative">
              <!-- 连接线 -->
              <div class="absolute left-2.5 top-0 bottom-2 w-px bg-gray-200 dark:bg-gray-700/50"></div>

              <li v-for="(childCatalog, index2) in catalog.children" :key="index2">
                <router-link
                  :to="{ path: '/wiki/' + route.params.wikiId, query: { articleId: childCatalog.articleId } }" :class="[
                    childCatalog.articleId == route.query.articleId
                      ? 'text-sky-600 bg-sky-50 font-semibold dark:text-sky-400 dark:bg-sky-900/20 border-r-2 border-sky-500'
                      : 'text-gray-500 hover:text-gray-900 hover:bg-gray-100 dark:text-gray-400 dark:hover:text-gray-200 dark:hover:bg-gray-800'
                  ]"
                  class="relative flex items-center px-3 py-2 text-sm rounded-l-lg cursor-pointer transition-all duration-200 truncate group"
                  @click="showMobileCatalog = false"> <!-- 移动端点击后自动关闭抽屉 -->
                  <span class="truncate" v-html="childCatalog.title"></span>
                </router-link>
              </li>
            </ul>
          </div>
        </div>
      </aside>

      <!-- 侧边栏切换按钮 (桌面端悬浮) -->
      <button :title="isExpand ? '收起目录' : '展开目录'"
        class="fixed bottom-8 left-6 z-30 p-3 bg-white dark:bg-gray-800 border border-gray-200 dark:border-gray-700 rounded-full shadow-lg hover:shadow-xl hover:bg-gray-50 dark:hover:bg-gray-700 transition-all duration-300 text-gray-500 dark:text-gray-400 hidden lg:flex items-center justify-center group"
        @click="shrinkAndExpand">
        <svg :class="{ 'rotate-180': !isExpand }" class="w-5 h-5 transition-transform duration-300" fill="none"
          stroke="currentColor" viewBox="0 0 24 24">
          <path d="M11 19l-7-7 7-7m8 14l-7-7 7-7" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" />
        </svg>
      </button>

      <!-- 移动端目录悬浮按钮 -->
      <button
        class="fixed bottom-20 right-6 z-30 p-3 bg-sky-500 text-white rounded-full shadow-lg hover:shadow-sky-500/30 transition-all duration-300 lg:hidden flex items-center justify-center"
        @click="showMobileCatalog = true">
        <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 6h16M4 12h16M4 18h7" />
        </svg>
      </button>

      <!-- 中间：文章内容区 -->
      <div class="flex-1 min-w-0 transition-all duration-300">

        <div class="max-w-4xl mx-auto">

          <!-- 加载状态：骨架屏 -->
          <div v-if="loading"
            class="bg-white dark:bg-gray-800 rounded-2xl shadow-sm border border-gray-100 dark:border-gray-700 p-8 min-h-[80vh] animate-pulse">
            <div class="h-10 bg-gray-200 dark:bg-gray-700 rounded w-3/4 mb-6"></div>
            <div class="flex gap-4 mb-10">
              <div class="h-6 bg-gray-200 dark:bg-gray-700 rounded w-24"></div>
              <div class="h-6 bg-gray-200 dark:bg-gray-700 rounded w-32 ml-auto"></div>
            </div>
            <div class="space-y-4">
              <div class="h-4 bg-gray-200 dark:bg-gray-700 rounded w-full"></div>
              <div class="h-4 bg-gray-200 dark:bg-gray-700 rounded w-full"></div>
              <div class="h-4 bg-gray-200 dark:bg-gray-700 rounded w-5/6"></div>
              <div class="h-4 bg-gray-200 dark:bg-gray-700 rounded w-full"></div>
              <div class="h-4 bg-gray-200 dark:bg-gray-700 rounded w-4/5"></div>
            </div>
          </div>

          <article v-else
            class="bg-white dark:bg-gray-800 rounded-2xl shadow-sm border border-gray-100 dark:border-gray-700 overflow-hidden min-h-[80vh] transition-all duration-300">

            <!-- 文章头部 -->
            <div class="px-6 md:px-10 pt-10 pb-6 border-b border-gray-100 dark:border-gray-700/50">
              <h1 class="text-3xl md:text-4xl font-extrabold text-gray-900 dark:text-white mb-6 leading-tight">
                {{ article.title }}
              </h1>

              <!-- Meta 信息 -->
              <div class="flex flex-wrap items-center gap-4 text-sm text-gray-500 dark:text-gray-400">
                <div class="flex items-center bg-gray-50 dark:bg-gray-700/30 px-3 py-1.5 rounded-full">
                  <svg class="w-4 h-4 mr-1.5 text-gray-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path d="M8 7V3m8 4V3m-9 8h10M5 21h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v12a2 2 0 002 2z"
                      stroke-linecap="round" stroke-linejoin="round" stroke-width="2" />
                  </svg>
                  <span class="hidden md:inline">发布于 </span>{{ article.createTime }}
                </div>
                <div class="flex items-center gap-4 ml-auto">
                  <span class="flex items-center" title="阅读量">
                    <svg class="w-4 h-4 mr-1 text-sky-500" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                      <path d="M15 12a3 3 0 11-6 0 3 3 0 016 0z" stroke-linecap="round" stroke-linejoin="round"
                        stroke-width="2" />
                      <path
                        d="M2.458 12C3.732 7.943 7.523 5 12 5c4.478 0 8.268 2.943 9.542 7-1.274 4.057-5.064 7-9.542 7-4.477 0-8.268-2.943-9.542-7z"
                        stroke-linecap="round" stroke-linejoin="round" stroke-width="2" />
                    </svg>
                    {{ article.readNum }}
                  </span>
                  <span class="flex items-center" title="字数">
                    <svg class="w-4 h-4 mr-1 text-emerald-500" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                      <path
                        d="M9 12h6m-6 4h6m2 5H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z"
                        stroke-linecap="round" stroke-linejoin="round" stroke-width="2" />
                    </svg>
                    {{ article.totalWords }} 字
                  </span>
                  <span class="flex items-center" title="阅读时长">
                    <svg class="w-4 h-4 mr-1 text-rose-500" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                      <path d="M12 8v4l3 3m6-3a9 9 0 11-18 0 9 9 0 0118 0z" stroke-linecap="round"
                        stroke-linejoin="round" stroke-width="2" />
                    </svg>
                    {{ article.readTime }}
                  </span>
                </div>
              </div>
            </div>

            <!-- 正文内容 -->
            <div class="px-6 md:px-10 py-10">
              <div :class="{ 'dark': isDark }">
                <div v-viewer class="article-content prose prose-lg dark:prose-invert max-w-none"
                  v-html="article.content"></div>
              </div>
            </div>

            <!-- 底部更新时间 -->
            <div
              class="px-6 md:px-10 py-6 border-t border-gray-100 dark:border-gray-700 bg-gray-50/50 dark:bg-gray-800/50">
              <div class="flex items-center text-sm text-gray-400 italic">
                <svg class="w-4 h-4 mr-1.5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path
                    d="M11 5H6a2 2 0 00-2 2v11a2 2 0 002 2h11a2 2 0 002-2v-5m-1.414-9.414a2 2 0 112.828 2.828L11.828 15H9v-2.828l8.586-8.586z"
                    stroke-linecap="round" stroke-linejoin="round" stroke-width="2" />
                </svg>
                最后更新于：{{ article.updateTime }}
              </div>
            </div>

          </article>

          <!-- 上下篇导航 -->
          <nav v-if="!loading && preNext" class="grid grid-cols-1 md:grid-cols-2 gap-4 mt-8">
            <!-- 上一篇 -->
            <div v-if="preNext.preArticle"
              class="group p-5 bg-white dark:bg-gray-800 border border-gray-100 dark:border-gray-700 rounded-xl cursor-pointer hover:border-sky-300 dark:hover:border-sky-600 hover:shadow-lg hover:-translate-y-1 transition-all duration-300 relative overflow-hidden"
              @click="goWikiArticleDetailPage(preNext.preArticle.articleId)">
              <div
                class="absolute top-0 left-0 w-1 h-full bg-sky-500 opacity-0 group-hover:opacity-100 transition-opacity">
              </div>
              <div
                class="flex items-center gap-2 text-xs font-bold uppercase tracking-wider text-gray-400 mb-2 group-hover:text-sky-600 dark:group-hover:text-sky-400 transition-colors">
                <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path d="M15 19l-7-7 7-7" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" />
                </svg>
                上一篇
              </div>
              <div
                class="text-base font-medium text-gray-700 dark:text-gray-200 truncate group-hover:text-sky-600 dark:group-hover:text-sky-400 transition-colors"
                v-html="preNext.preArticle.articleTitle"></div>
            </div>
            <div v-else class="hidden md:block"></div>

            <!-- 下一篇 -->
            <div v-if="preNext.nextArticle"
              class="group p-5 bg-white dark:bg-gray-800 border border-gray-100 dark:border-gray-700 rounded-xl cursor-pointer hover:border-sky-300 dark:hover:border-sky-600 hover:shadow-lg hover:-translate-y-1 transition-all duration-300 text-right relative overflow-hidden"
              @click="goWikiArticleDetailPage(preNext.nextArticle.articleId)">
              <div
                class="absolute top-0 right-0 w-1 h-full bg-sky-500 opacity-0 group-hover:opacity-100 transition-opacity">
              </div>
              <div
                class="flex items-center justify-end gap-2 text-xs font-bold uppercase tracking-wider text-gray-400 mb-2 group-hover:text-sky-600 dark:group-hover:text-sky-400 transition-colors">
                下一篇
                <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path d="M9 5l7 7-7 7" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" />
                </svg>
              </div>
              <div
                class="text-base font-medium text-gray-700 dark:text-gray-200 truncate group-hover:text-sky-600 dark:group-hover:text-sky-400 transition-colors"
                v-html="preNext.nextArticle.articleTitle"></div>
            </div>
          </nav>

        </div>
      </div>

      <!-- 右边栏：文章大纲 (WikiToc) -->
      <div class="hidden lg:block w-64 shrink-0">
        <WikiToc></WikiToc>
      </div>

    </main>

    <WikiFooter></WikiFooter>

    <!-- 返回顶部 -->
    <ScrollToTopButton></ScrollToTopButton>
  </div>
</template>

<script setup>
import WikiHeader from '@/layouts/frontend/components/WikiHeader.vue'
import WikiFooter from '@/layouts/frontend/components/WikiFooter.vue'
import { nextTick, ref, watch } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { getArticleDetail } from '@/api/frontend/article'
import { useDark } from '@vueuse/core'
import hljs from 'highlight.js'
import 'highlight.js/styles/tokyo-night-dark.css'
import { getWikiArticlePreNext, getWikiCatalogs } from "@/api/frontend/wiki.js";
import ScrollToTopButton from "@/layouts/frontend/components/ScrollToTopButton.vue";
import WikiToc from "@/layouts/frontend/components/WikiToc.vue";

const route = useRoute()
const router = useRouter()
const isDark = useDark()

const article = ref({})
const preNext = ref(null)
const catalogs = ref([])
const collapsedCatalogs = ref([]) // 记录收起的目录ID
const loading = ref(true)
const showMobileCatalog = ref(false)

// 获取目录数据
getWikiCatalogs(route.params.wikiId).then(res => {
  if (res.success) {
    catalogs.value = res.data
    // 默认展开所有或根据需求处理
  }
})

// 控制目录展开/收起 (手风琴效果)
const toggleCatalog = (id) => {
  const index = collapsedCatalogs.value.indexOf(id)
  if (index > -1) {
    collapsedCatalogs.value.splice(index, 1)
  } else {
    collapsedCatalogs.value.push(id)
  }
}

// 侧边栏展开状态
const isExpand = ref(true)
const shrinkAndExpand = () => {
  isExpand.value = !isExpand.value
}

// 获取文章详情
function refreshArticleDetail(articleId) {
  if (!articleId) return

  loading.value = true
  showMobileCatalog.value = false // 切换文章时关闭移动端菜单

  getArticleDetail(articleId).then((res) => {
    if (!res.success && res.errorCode === '20010') {
      router.push({ name: 'NotFound' })
      return
    }

    article.value = res.data

    nextTick(() => {
      // 代码高亮
      document.querySelectorAll('.article-content pre code').forEach((block) => {
        hljs.highlightElement(block)
      })

      // 代码复制按钮 (复用 article-detail 的逻辑)
      addCopyButtons()
    })
  }).finally(() => {
    // 延迟一点取消loading，避免闪烁
    setTimeout(() => {
      loading.value = false
    }, 300)
  })

  // 获取上下篇
  getWikiArticlePreNext({ id: route.params.wikiId, articleId: articleId }).then(res => {
    if (res.success) {
      preNext.value = res.data
    }
  })
}

// 添加代码复制按钮
const addCopyButtons = () => {
  document.querySelectorAll('.article-content pre').forEach(preElement => {
    if (preElement.querySelector('.copy-code-btn')) return;

    let firstCode = preElement.querySelector('code');
    if (firstCode) {
      let copyCodeBtn = document.createElement('button');
      copyCodeBtn.className = 'copy-code-btn absolute top-3 right-3 p-2 rounded-md bg-white/10 hover:bg-white/20 text-gray-300 hover:text-white transition-all opacity-0 group-hover:opacity-100 focus:outline-none backdrop-blur-sm';
      copyCodeBtn.innerHTML = `<svg class="w-4 h-4" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 16H6a2 2 0 01-2-2V6a2 2 0 012-2h8a2 2 0 012 2v2m-6 12h8a2 2 0 002-2v-8a2 2 0 00-2-2h-8a2 2 0 00-2 2v8a2 2 0 002 2z" /></svg>`;

      preElement.classList.add('group', 'relative');
      preElement.appendChild(copyCodeBtn);

      copyCodeBtn.addEventListener('click', () => {
        copyToClipboard(preElement.textContent);
        const originalIcon = copyCodeBtn.innerHTML;
        copyCodeBtn.innerHTML = `<svg class="w-4 h-4 text-emerald-400" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7" /></svg>`;
        setTimeout(() => copyCodeBtn.innerHTML = originalIcon, 2000);
      });
    }
  })
}

function copyToClipboard(text) {
  if (navigator.clipboard && window.isSecureContext) {
    navigator.clipboard.writeText(text);
  } else {
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

const goWikiArticleDetailPage = (articleId) => {
  router.push({ path: '/wiki/' + route.params.wikiId, query: { articleId } })
}

refreshArticleDetail(route.query.articleId)

watch(route, (newRoute) => {
  // 只有当文章ID变化时才刷新
  if (newRoute.query.articleId !== article.value.id) {
    refreshArticleDetail(newRoute.query.articleId)
  }
})
</script>

<style scoped>
/* 滚动条美化 */
.custom-scrollbar::-webkit-scrollbar {
  width: 4px;
}

.custom-scrollbar::-webkit-scrollbar-track {
  background: transparent;
}

.custom-scrollbar::-webkit-scrollbar-thumb {
  background-color: #e5e7eb;
  border-radius: 20px;
}

.dark .custom-scrollbar::-webkit-scrollbar-thumb {
  background-color: #374151;
}

/* 排版样式 */
::v-deep(.article-content) {
  font-family: 'Inter', system-ui, -apple-system, sans-serif;
  line-height: 1.8;
  color: #374151;
  font-size: 1.05rem;
}

::v-deep(.article-content h1),
::v-deep(.article-content h2),
::v-deep(.article-content h3),
::v-deep(.article-content h4) {
  color: #111827;
  font-weight: 700;
  margin-top: 2em;
  margin-bottom: 0.8em;
  line-height: 1.3;
  letter-spacing: -0.025em;
}

::v-deep(.article-content h1) {
  font-size: 2.25em;
}

::v-deep(.article-content h2) {
  font-size: 1.75em;
  padding-bottom: 0.3em;
  border-bottom: 1px solid #e5e7eb;
}

::v-deep(.article-content h3) {
  font-size: 1.5em;
}

::v-deep(.article-content h4) {
  font-size: 1.25em;
}

::v-deep(.article-content p) {
  margin: 1.25em 0;
  text-align: justify;
}

::v-deep(.article-content ul),
::v-deep(.article-content ol) {
  margin: 1.25em 0;
  padding-left: 1.625em;
}

::v-deep(.article-content li) {
  margin: 0.5em 0;
}

::v-deep(.article-content ul) {
  list-style-type: disc;
}

::v-deep(.article-content ol) {
  list-style-type: decimal;
}

::v-deep(.article-content blockquote) {
  border-left: 4px solid #3b82f6;
  background: #eff6ff;
  padding: 1rem 1.5rem;
  margin: 1.5em 0;
  border-radius: 0.5rem;
  color: #1e40af;
  font-style: italic;
}

::v-deep(.article-content pre) {
  background-color: #1e293b !important;
  color: #e2e8f0;
  border-radius: 0.75rem;
  padding: 1.25em;
  margin: 1.5em 0;
  overflow-x: auto;
  box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
  font-size: 0.9em;
}

::v-deep(.article-content code) {
  color: #ea580c;
  background-color: #ffedd5;
  padding: 0.2em 0.4em;
  border-radius: 0.25rem;
  font-size: 0.875em;
  font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace;
}

::v-deep(.article-content pre code) {
  color: inherit;
  background-color: transparent;
  padding: 0;
  font-size: inherit;
}

::v-deep(.article-content img) {
  border-radius: 0.75rem;
  box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1);
  margin: 2em auto;
  max-width: 100%;
  display: block;
}

::v-deep(.article-content a) {
  color: #0ea5e9;
  text-decoration: none;
  border-bottom: 1px solid transparent;
  transition: border-color 0.2s;
  font-weight: 500;
}

::v-deep(.article-content a:hover) {
  border-bottom-color: #0ea5e9;
}

::v-deep(.article-content table) {
  width: 100%;
  border-collapse: collapse;
  margin: 2em 0;
}

::v-deep(.article-content th),
::v-deep(.article-content td) {
  border: 1px solid #e5e7eb;
  padding: 0.75em;
  text-align: left;
}

::v-deep(.article-content th) {
  background-color: #f9fafb;
  font-weight: 600;
}

/* 暗黑模式适配 */
::v-deep(.dark .article-content) {
  color: #d1d5db;
}

::v-deep(.dark .article-content h1),
::v-deep(.dark .article-content h2),
::v-deep(.dark .article-content h3),
::v-deep(.dark .article-content h4) {
  color: #f3f4f6;
}

::v-deep(.dark .article-content h2) {
  border-bottom-color: #374151;
}

::v-deep(.dark .article-content blockquote) {
  background: #1e293b;
  border-left-color: #60a5fa;
  color: #93c5fd;
}

::v-deep(.dark .article-content code) {
  color: #fdba74;
  background-color: #7c2d12;
}

::v-deep(.dark .article-content a) {
  color: #38bdf8;
}

::v-deep(.dark .article-content th),
::v-deep(.dark .article-content td) {
  border-color: #374151;
}

::v-deep(.dark .article-content th) {
  background-color: #1f2937;
}
</style>
