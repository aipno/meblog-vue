<template>
  <div v-if="titles && titles.length > 0"
       :class="[currScrollY > 0 ? 'top-4' : 'top-[5.5rem]']"
       class="fixed right-[max(0px,calc(45%-48rem))] w-[19rem] transition-all duration-300 z-10 hidden xl:block">

    <div
        class="bg-white border border-gray-100 rounded-xl shadow-sm dark:bg-gray-800 dark:border-gray-700 overflow-hidden">
      <!-- 目录标题 -->
      <div class="p-4 border-b border-gray-100 dark:border-gray-700 bg-gray-50/50 dark:bg-gray-800/50">
        <h2 class="flex items-center text-sm font-bold text-gray-700 uppercase dark:text-gray-200">
          <!-- 目录图标 -->
          <svg class="inline icon w-4 h-4 mr-2 text-sky-500" viewBox="0 0 1024 1024"
               xmlns="http://www.w3.org/2000/svg">
            <path
                d="M857.6 25.6a76.8 76.8 0 0 1 76.8 76.8v819.2a76.8 76.8 0 0 1-76.8 76.8H166.4a76.8 76.8 0 0 1-76.8-76.8V102.4a76.8 76.8 0 0 1 76.8-76.8h691.2z m-102.4 678.4H473.6l-2.2528 0.064a38.4 38.4 0 0 0 0 76.672L473.6 780.8h281.6l2.2528-0.064a38.4 38.4 0 0 0 0-76.672L755.2 704z m0-230.4H473.6l-2.2528 0.064a38.4 38.4 0 0 0 0 76.672L473.6 550.4h281.6l2.2528-0.064a38.4 38.4 0 0 0 0-76.672L755.2 473.6z m0-230.4H473.6l-2.2528 0.064a38.4 38.4 0 0 0 0 76.672L473.6 320h281.6l2.2528-0.064a38.4 38.4 0 0 0 0-76.672L755.2 243.2z"
                fill="currentColor"></path>
          </svg>
          本文目录
        </h2>
      </div>

      <!-- 目录内容容器 -->
      <div class="toc-wrapper max-h-[calc(100vh-200px)] overflow-y-auto custom-scrollbar p-2">
        <ul class="toc space-y-0.5">
          <!-- 一级标题 (h2) -->
          <li v-for="(h2, index) in titles" :key="index">
            <div
                :class="[
                h2.index === activeHeadingIndex
                  ? 'text-sky-600 bg-sky-50 font-medium dark:bg-sky-900/20 dark:text-sky-400'
                  : 'text-gray-600 hover:bg-gray-100 hover:text-gray-900 dark:text-gray-400 dark:hover:bg-gray-700/50 dark:hover:text-gray-200'
              ]"
                :title="h2.text"
                class="px-3 py-1.5 text-sm rounded-md cursor-pointer transition-colors duration-200 select-none truncate block"
                @click="scrollToView(h2.offsetTop)">
              {{ h2.text }}
            </div>

            <!-- 二级标题 (h3) -->
            <template v-if="h2.children && h2.children.length > 0">
              <ul class="mt-0.5 space-y-0.5 border-l border-gray-100 dark:border-gray-700 ml-3 pl-1">
                <li v-for="(child, childIndex) in h2.children" :key="childIndex">
                  <div
                      :class="[
                      child.index === activeHeadingIndex
                        ? 'text-sky-600 bg-sky-50 font-medium dark:bg-sky-900/20 dark:text-sky-400 border-l-2 border-sky-500 -ml-[5px] pl-[13px]'
                        : 'text-gray-500 hover:bg-gray-100 hover:text-gray-900 dark:text-gray-500 dark:hover:bg-gray-700/50 dark:hover:text-gray-300'
                    ]"
                      :title="child.text"
                      class="px-3 py-1.5 text-xs rounded-md cursor-pointer transition-all duration-200 select-none truncate block"
                      @click="scrollToView(child.offsetTop)">
                    {{ child.text }}
                  </div>

                  <!-- 三级标题 (h4及以下) -->
                  <template v-if="child.children && child.children.length > 0">
                    <ul class="mt-0.5 space-y-0.5 border-l border-gray-100 dark:border-gray-700 ml-3 pl-1">
                      <li v-for="(grandChild, grandChildIndex) in child.children" :key="grandChildIndex">
                        <div
                            :class="[
                              grandChild.index === activeHeadingIndex
                                ? 'text-sky-600 bg-sky-50 font-medium dark:bg-sky-900/20 dark:text-sky-400 border-l-2 border-sky-500 -ml-[5px] pl-[13px]'
                                : 'text-gray-400 hover:bg-gray-100 hover:text-gray-900 dark:text-gray-500 dark:hover:bg-gray-700/50 dark:hover:text-gray-300'
                            ]"
                            :title="grandChild.text"
                            class="px-3 py-1.5 text-xs rounded-md cursor-pointer transition-all duration-200 select-none truncate block"
                            @click="scrollToView(grandChild.offsetTop)">
                          {{ grandChild.text }}
                        </div>
                      </li>
                    </ul>
                  </template>
                </li>
              </ul>
            </template>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script setup>
import {onBeforeUnmount, onMounted, ref} from 'vue'
import {useDark} from '@vueuse/core'

// 是否是暗黑模式
const isDark = useDark()

// 响应式的目录数据
const titles = ref([])
onMounted(() => {
  // 通过 .article-content 样式来获取父级 div
  const container = document.querySelector('.article-content')

  // 使用 MutationObserver 监视 DOM 的变化
  const observer = new MutationObserver(mutationsList => {
    for (let mutation of mutationsList) {
      if (mutation.type === 'childList') {
        // 先清空目录缓存数据
        titles.value = []
        // 计算目录数据
        initTocData(container)

        // 监听所有图片的加载事件
        const images = container.querySelectorAll('img');
        images.forEach(img => {
          img.addEventListener('load', () => {
            // 图片加载完成后重新计算标题的 offsetTop
            initTocData(container)
          })
        })

        // 添加滚动事件监听
        window.addEventListener('scroll', handleContentScroll);
      }
    }
  })

  // 配置监视子节点的变化
  const config = {childList: true, subtree: true}
  // 开始观察正文 div 的内容变化
  observer.observe(container, config)
})

// 记录当前被选中的标题下标
const activeHeadingIndex = ref(-1)

// 当前 Y 轴滚动的偏移量
const currScrollY = ref(0)

// 处理滚动事件
function handleContentScroll() {
  // 当前的滚动位置
  let scrollY = window.scrollY
  currScrollY.value = scrollY

  // 递归检查所有标题
  function checkTitles(titlesArray) {
    titlesArray.forEach(title => {
      let offsetTop = title.offsetTop
      // 增加一点偏移量，使选中更自然
      if (scrollY >= offsetTop) {
        activeHeadingIndex.value = title.index
      }

      // 递归检查子标题
      if (title.children && title.children.length > 0) {
        checkTitles(title.children)
      }
    })
  }

  checkTitles(titles.value)
}

// 移除滚动监听
onBeforeUnmount(() => window.removeEventListener('scroll', handleContentScroll))

// 滚动到指定的位置
function scrollToView(offsetTop) {
  // 增加偏移量，避免标题被 Header 遮挡
  window.scrollTo({top: offsetTop + 80, behavior: "smooth"});
}

// 初始化标题数据
function initTocData(container) {
  // 提取标题
  let levels = ['h2', 'h3', 'h4', 'h5', 'h6']
  let headings = container.querySelectorAll(levels)

  // 存放组装后的目录标题数据
  let titlesArr = []

  // 下标
  let index = 1
  // 用于构建层级关系的栈
  let stack = []

  headings.forEach(heading => {
    // 标题等级， h2 -> 级别 2 ； h3 -> 级别3，以此类推
    let headingLevel = parseInt(heading.tagName.substring(1))
    // 标题文字
    let headingText = heading.innerText
    // 标题的位置（距离顶部的距离）
    // 调整 offsetTop 计算，使其更准确
    let offsetTop = heading.offsetTop - 95

    // 创建当前标题节点
    let currentNode = {
      index,
      level: headingLevel,
      text: headingText,
      offsetTop,
      children: []
    }

    // 处理层级关系
    if (stack.length === 0) {
      // 栈为空，直接添加到根目录
      titlesArr.push(currentNode)
      stack.push(currentNode)
    } else {
      // 找到合适的父级标题
      while (stack.length > 0 && stack[stack.length - 1].level >= headingLevel) {
        stack.pop()
      }

      if (stack.length > 0) {
        // 添加到父级的children中
        stack[stack.length - 1].children.push(currentNode)
      } else {
        // 没有合适的父级，添加到根目录
        titlesArr.push(currentNode)
      }

      stack.push(currentNode)
    }

    // 下标 +1
    index++
  })

  // 设置数据
  titles.value = titlesArr
}
</script>

<style scoped>
/* 自定义滚动条样式 */
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
  background-color: #4b5563;
}
</style>