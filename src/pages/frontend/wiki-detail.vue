<template>
  <div class="main min-h-screen flex flex-col bg-gray-50 dark:bg-[#0d1117] transition-colors duration-300 font-sans">
    <WikiHeader></WikiHeader>

    <main class="grow container max-w-screen-3xl mx-auto px-4 sm:px-6 md:px-8 py-6 relative">

      <!-- 左侧：知识库目录 (Sidebar) -->
      <!-- 桌面端固定侧边栏 -->
      <aside
          :class="[isExpand ? 'translate-x-0' : '-translate-x-full']"
          class="fixed top-[4rem] bottom-0 left-0 z-20 w-72 pt-6 pb-8 bg-white border-r border-gray-200 dark:bg-[#0d1117] dark:border-gray-800 overflow-y-auto custom-scrollbar transition-transform duration-300 ease-in-out hidden lg:block"
      >
        <div class="px-4 mb-4">
          <h3 class="text-sm font-bold text-gray-500 uppercase tracking-wider dark:text-gray-400">目录结构</h3>
        </div>

        <!-- 目录列表 -->
        <div class="space-y-1 px-3">
          <div v-for="(catalog, index) in catalogs" :key="index" class="space-y-1">
            <!-- 一级目录 -->
            <div
                class="group flex items-center justify-between px-3 py-2 text-sm font-medium text-gray-700 dark:text-gray-200 rounded-lg hover:bg-gray-100 dark:hover:bg-gray-800 cursor-pointer transition-colors"
                @click="toggleCatalog(catalog.id)">
              <span class="truncate" v-html="catalog.title"></span>
              <svg :class="{'rotate-90': !collapsedCatalogs.includes(catalog.id)}"
                   class="w-4 h-4 text-gray-400 transition-transform duration-200"
                   fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path d="M9 5l7 7-7 7" stroke-linecap="round" stroke-linejoin="round" stroke-width="2"/>
              </svg>
            </div>

            <!-- 二级目录 (展开/收起) -->
            <ul v-show="!collapsedCatalogs.includes(catalog.id)" class="pl-4 space-y-1 relative">
              <!-- 连接线 -->
              <div class="absolute left-2.5 top-0 bottom-2 w-px bg-gray-200 dark:bg-gray-700"></div>

              <li v-for="(childCatalog, index2) in catalog.children" :key="index2">
                <a :class="[
                                childCatalog.articleId == route.query.articleId
                                ? 'text-sky-600 bg-sky-50 font-semibold dark:text-sky-400 dark:bg-sky-900/20'
                                : 'text-gray-500 hover:text-gray-900 hover:bg-gray-100 dark:text-gray-400 dark:hover:text-gray-200 dark:hover:bg-gray-800'
                            ]"
                   class="relative flex items-center px-3 py-2 text-sm rounded-lg cursor-pointer transition-colors truncate"
                   @click="goWikiArticleDetailPage(childCatalog.articleId)">
                  <!-- 选中指示点 -->
                  <span v-if="childCatalog.articleId == route.query.articleId"
                        class="absolute left-[-5px] top-1/2 -translate-y-1/2 w-1.5 h-1.5 rounded-full bg-sky-500"></span>
                  <span class="truncate" v-html="childCatalog.title"></span>
                </a>
              </li>
            </ul>
          </div>
        </div>
      </aside>

      <!-- 移动端目录抽屉 (可选，这里暂用简单的显示隐藏逻辑或复用上方逻辑，此处保持PC优化重点) -->

      <!-- 侧边栏切换按钮 (悬浮) -->
      <button
          :title="isExpand ? '收起目录' : '展开目录'"
          class="fixed bottom-8 left-6 z-30 p-3 bg-white dark:bg-gray-800 border border-gray-200 dark:border-gray-700 rounded-full shadow-lg hover:shadow-xl hover:bg-gray-50 dark:hover:bg-gray-700 transition-all duration-300 text-gray-500 dark:text-gray-400 hidden lg:flex items-center justify-center group"
          @click="shrinkAndExpand">
        <svg :class="{'rotate-180': !isExpand}" class="w-5 h-5 transition-transform duration-300" fill="none"
             stroke="currentColor" viewBox="0 0 24 24">
          <path d="M11 19l-7-7 7-7m8 14l-7-7 7-7" stroke-linecap="round" stroke-linejoin="round" stroke-width="2"/>
        </svg>
      </button>

      <!-- 中间：文章内容区 -->
      <div :class="[isExpand ? 'lg:pl-72' : 'lg:pl-0', 'xl:pr-[19.5rem]']"
           class="transition-all duration-300 ease-in-out"> <!-- xl:pr 留出右侧 TOC 空间 -->

        <div class="max-w-4xl mx-auto">
          <!-- 面包屑/导航辅助 (可选) -->

          <article
              class="bg-white dark:bg-gray-800 rounded-2xl shadow-sm border border-gray-100 dark:border-gray-700 overflow-hidden min-h-[80vh]">

            <!-- 文章头部 -->
            <div class="px-8 pt-10 pb-6 border-b border-gray-100 dark:border-gray-700">
              <h1 class="text-3xl md:text-4xl font-extrabold text-gray-900 dark:text-white mb-6 leading-tight">
                {{ article.title }}
              </h1>

              <!-- Meta 信息 -->
              <div class="flex flex-wrap items-center gap-4 text-sm text-gray-500 dark:text-gray-400">
                <div class="flex items-center bg-gray-100 dark:bg-gray-700/50 px-2.5 py-1 rounded-md">
                  <svg class="w-4 h-4 mr-1.5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path d="M8 7V3m8 4V3m-9 8h10M5 21h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v12a2 2 0 002 2z"
                          stroke-linecap="round" stroke-linejoin="round"
                          stroke-width="2"/>
                  </svg>
                  发布于 {{ article.createTime }}
                </div>
                <div class="flex items-center gap-4 ml-auto">
                      <span class="flex items-center" title="阅读量">
                          <svg class="w-4 h-4 mr-1" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path
                              d="M15 12a3 3 0 11-6 0 3 3 0 016 0z" stroke-linecap="round" stroke-linejoin="round"
                              stroke-width="2"/><path
                              d="M2.458 12C3.732 7.943 7.523 5 12 5c4.478 0 8.268 2.943 9.542 7-1.274 4.057-5.064 7-9.542 7-4.477 0-8.268-2.943-9.542-7z"
                              stroke-linecap="round"
                              stroke-linejoin="round"
                              stroke-width="2"/></svg>
                          {{ article.readNum }}
                      </span>
                  <span class="flex items-center" title="字数">
                          <svg class="w-4 h-4 mr-1" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path
                              d="M9 12h6m-6 4h6m2 5H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z"
                              stroke-linecap="round" stroke-linejoin="round"
                              stroke-width="2"/></svg>
                          {{ article.totalWords }} 字
                      </span>
                  <span class="flex items-center" title="阅读时长">
                          <svg class="w-4 h-4 mr-1" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path
                              d="M12 8v4l3 3m6-3a9 9 0 11-18 0 9 9 0 0118 0z" stroke-linecap="round"
                              stroke-linejoin="round"
                              stroke-width="2"/></svg>
                          {{ article.readTime }}
                      </span>
                </div>
              </div>
            </div>

            <!-- 正文内容 -->
            <div class="px-8 py-8">
              <div :class="{ 'dark': isDark }">
                <div v-viewer class="article-content" v-html="article.content"></div>
              </div>
            </div>

            <!-- 底部更新时间 -->
            <div class="px-8 py-6 border-t border-gray-100 dark:border-gray-700 bg-gray-50/50 dark:bg-gray-800/50">
              <div class="flex items-center text-sm text-gray-400 italic">
                <svg class="w-4 h-4 mr-1.5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path
                      d="M11 5H6a2 2 0 00-2 2v11a2 2 0 002 2h11a2 2 0 002-2v-5m-1.414-9.414a2 2 0 112.828 2.828L11.828 15H9v-2.828l8.586-8.586z"
                      stroke-linecap="round" stroke-linejoin="round"
                      stroke-width="2"/>
                </svg>
                最后更新于：{{ article.updateTime }}
              </div>
            </div>

          </article>

          <!-- 上下篇导航 -->
          <nav v-if="preNext" class="grid grid-cols-1 md:grid-cols-2 gap-4 mt-6">
            <!-- 上一篇 -->
            <div v-if="preNext.preArticle"
                 class="group p-4 bg-white dark:bg-gray-800 border border-gray-100 dark:border-gray-700 rounded-xl cursor-pointer hover:border-sky-300 dark:hover:border-sky-600 hover:shadow-md transition-all"
                 @click="goWikiArticleDetailPage(preNext.preArticle.articleId)">
              <div
                  class="flex items-center gap-2 text-xs text-gray-400 mb-1 group-hover:text-sky-600 dark:group-hover:text-sky-400 transition-colors">
                <svg class="w-3 h-3" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path d="M15 19l-7-7 7-7" stroke-linecap="round" stroke-linejoin="round" stroke-width="2"/>
                </svg>
                上一篇
              </div>
              <div class="text-sm font-medium text-gray-700 dark:text-gray-200 truncate"
                   v-html="preNext.preArticle.articleTitle"></div>
            </div>
            <div v-else class="hidden md:block"></div>

            <!-- 下一篇 -->
            <div v-if="preNext.nextArticle"
                 class="group p-4 bg-white dark:bg-gray-800 border border-gray-100 dark:border-gray-700 rounded-xl cursor-pointer hover:border-sky-300 dark:hover:border-sky-600 hover:shadow-md transition-all text-right"
                 @click="goWikiArticleDetailPage(preNext.nextArticle.articleId)">
              <div
                  class="flex items-center justify-end gap-2 text-xs text-gray-400 mb-1 group-hover:text-sky-600 dark:group-hover:text-sky-400 transition-colors">
                下一篇
                <svg class="w-3 h-3" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path d="M9 5l7 7-7 7" stroke-linecap="round" stroke-linejoin="round" stroke-width="2"/>
                </svg>
              </div>
              <div class="text-sm font-medium text-gray-700 dark:text-gray-200 truncate"
                   v-html="preNext.nextArticle.articleTitle"></div>
            </div>
          </nav>

        </div>
      </div>

      <!-- 右边栏：文章大纲 (WikiToc) -->
      <!-- WikiToc 组件内部已优化为 fixed 定位，这里只需引入即可 -->
      <WikiToc></WikiToc>

    </main>

    <WikiFooter></WikiFooter>

    <!-- 返回顶部 -->
    <ScrollToTopButton></ScrollToTopButton>
  </div>
</template>

<script setup>
import WikiHeader from '@/layouts/frontend/components/WikiHeader.vue'
import WikiFooter from '@/layouts/frontend/components/WikiFooter.vue'
import {nextTick, ref, watch} from 'vue'
import {useRoute, useRouter} from 'vue-router'
import {getArticleDetail} from '@/api/frontend/article'
import {useDark} from '@vueuse/core'
import hljs from 'highlight.js'
import 'highlight.js/styles/tokyo-night-dark.css'
import {getWikiArticlePreNext, getWikiCatalogs} from "@/api/frontend/wiki.js";
import ScrollToTopButton from "@/layouts/frontend/components/ScrollToTopButton.vue";
import WikiToc from "@/layouts/frontend/components/WikiToc.vue";

const route = useRoute()
const router = useRouter()
const isDark = useDark()

const article = ref({})
const preNext = ref(null)
const catalogs = ref([])
const collapsedCatalogs = ref([]) // 记录收起的目录ID

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

  getArticleDetail(articleId).then((res) => {
    if (!res.success && res.errorCode === '20010') {
      router.push({name: 'NotFound'})
      return
    }

    article.value = res.data

    nextTick(() => {
      // 代码高亮
      document.querySelectorAll('pre code').forEach((block) => {
        hljs.highlightElement(block)
      })

      // 代码复制按钮 (复用 article-detail 的逻辑)
      addCopyButtons()
    })
  })

  // 获取上下篇
  getWikiArticlePreNext({id: route.params.wikiId, articleId: articleId}).then(res => {
    if (res.success) {
      preNext.value = res.data
    }
  })
}

// 添加代码复制按钮
const addCopyButtons = () => {
  document.querySelectorAll('pre').forEach(preElement => {
    if (preElement.querySelector('.copy-code-btn')) return;

    let firstCode = preElement.querySelector('code');
    if (firstCode) {
      let copyCodeBtn = document.createElement('button');
      copyCodeBtn.className = 'copy-code-btn absolute top-2 right-2 p-1.5 rounded-md bg-gray-700/50 hover:bg-gray-600 text-gray-300 hover:text-white transition-all opacity-0 group-hover:opacity-100 focus:outline-none';
      copyCodeBtn.innerHTML = `<svg class="w-4 h-4" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 16H6a2 2 0 01-2-2V6a2 2 0 012-2h8a2 2 0 012 2v2m-6 12h8a2 2 0 002-2v-8a2 2 0 00-2-2h-8a2 2 0 00-2 2v8a2 2 0 002 2z" /></svg>`;

      preElement.classList.add('group', 'relative');
      preElement.appendChild(copyCodeBtn);

      copyCodeBtn.addEventListener('click', () => {
        copyToClipboard(preElement.textContent);
        const originalIcon = copyCodeBtn.innerHTML;
        copyCodeBtn.innerHTML = `<svg class="w-4 h-4 text-green-400" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7" /></svg>`;
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
  router.push({path: '/wiki/' + route.params.wikiId, query: {articleId}})
}

refreshArticleDetail(route.query.articleId)

watch(route, (newRoute) => {
  refreshArticleDetail(newRoute.query.articleId)
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

/* 排版样式 (复用 Article Detail)
  注意：由于 Scoped 限制，这里需要重新声明 deep 样式，或者将排版样式提取为全局 CSS
  这里为了文件独立性，重新内联声明一遍核心样式
*/
::v-deep(.article-content) {
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, "Segoe UI", Helvetica, Arial, sans-serif;
  line-height: 1.8;
  color: #374151;
  font-size: 1.05rem;
}

::v-deep(.article-content h1), ::v-deep(.article-content h2), ::v-deep(.article-content h3) {
  color: #111827;
  font-weight: 700;
  margin-top: 2em;
  margin-bottom: 1em;
  line-height: 1.3;
}

::v-deep(.article-content h2) {
  font-size: 1.75em;
  padding-left: 12px;
  border-left: 4px solid #0284c7;
}

::v-deep(.article-content p) {
  margin: 1.25em 0;
  text-align: justify;
}

::v-deep(.article-content blockquote) {
  border-left: 4px solid #e5e7eb;
  background: #f9fafb;
  padding: 1rem;
  color: #4b5563;
  font-style: italic;
}

::v-deep(.article-content pre) {
  background-color: #1e293b !important;
  color: #e2e8f0;
  border-radius: 0.75rem;
  padding: 1.25em;
  margin: 1.5em 0;
  overflow-x: auto;
}

::v-deep(.article-content code) {
  color: #ef4444;
  background-color: #fee2e2;
  padding: 0.2em 0.4em;
  border-radius: 0.25rem;
  font-size: 0.875em;
}

::v-deep(.article-content pre code) {
  color: inherit;
  background-color: transparent;
  padding: 0;
}

::v-deep(.article-content img) {
  rounded: 0.75rem;
  box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
  margin: 2em auto;
  max-width: 100%;
  display: block;
}

::v-deep(.article-content a) {
  color: #0284c7;
  text-decoration: none;
  border-bottom: 1px solid transparent;
  transition: border-color 0.2s;
}

::v-deep(.article-content a:hover) {
  border-bottom-color: #0284c7;
}

/* 暗黑模式适配 */
::v-deep(.dark .article-content) {
  color: #d1d5db;
}

::v-deep(.dark .article-content h1), ::v-deep(.dark .article-content h2) {
  color: #f3f4f6;
}

::v-deep(.dark .article-content h2) {
  border-left-color: #38bdf8;
}

::v-deep(.dark .article-content blockquote) {
  background: #1f2937;
  border-left-color: #374151;
  color: #9ca3af;
}

::v-deep(.dark .article-content code) {
  color: #fca5a5;
  background-color: #7f1d1d;
}

::v-deep(.dark .article-content a) {
  color: #38bdf8;
}
</style>