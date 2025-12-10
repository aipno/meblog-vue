<script setup>
import {useTabList} from '@/composables/useTagList.js'
import {ArrowDown, CircleClose, Close} from "@element-plus/icons-vue";

const {menuStore, activeTab, tabList, tabChange, removeTab, handleCloseTab} = useTabList()
</script>

<template>
  <!-- 标签导航栏容器 -->
  <!-- transition-[left] 确保左侧菜单收缩时，标签栏左边距平滑过渡 -->
  <div
      :style="{left: menuStore.menuWidth}"
      class="fixed top-[64px] h-[44px] right-0 z-30 bg-white border-b border-gray-100 shadow-sm transition-[left] duration-300 flex items-center px-4 select-none">

    <!-- 标签页区域 -->
    <div class="flex-1 overflow-hidden h-full">
      <el-tabs
          v-model="activeTab"
          class="admin-tabs h-full"
          type="card"
          @tab-remove="removeTab"
          @tab-change="tabChange"
      >
        <el-tab-pane
            v-for="item in tabList"
            :key="item.path"
            :closable="item.path !== '/admin/index'"
            :label="item.title"
            :name="item.path"
        >
        </el-tab-pane>
      </el-tabs>
    </div>

    <!-- 右侧功能菜单 -->
    <div class="flex-shrink-0 ml-4 flex items-center">
      <el-dropdown trigger="click" @command="handleCloseTab">
        <span
            class="flex items-center justify-center w-8 h-8 rounded hover:bg-gray-100 cursor-pointer transition-colors text-gray-600">
            <el-icon size="16"><ArrowDown/></el-icon>
        </span>
        <template #dropdown>
          <el-dropdown-menu class="min-w-[120px]">
            <el-dropdown-item :icon="Close" command="closeOthers">关闭其他</el-dropdown-item>
            <el-dropdown-item :icon="CircleClose" command="closeAll">关闭全部</el-dropdown-item>
            <!-- 预留刷新功能，逻辑需在 useTabList 中实现，这里暂不绑定 -->
            <!-- <el-dropdown-item command="refresh" :icon="RefreshLeft" divided>刷新当前</el-dropdown-item> -->
          </el-dropdown-menu>
        </template>
      </el-dropdown>
    </div>
  </div>

  <!-- 占位 div，防止内容被 fixed 头部遮挡 -->
  <div class="h-[44px]"></div>
</template>

<style scoped>
/* 深度选择器覆盖 Element Plus Tabs 默认样式 */

/* 1. 隐藏底部默认的灰色线条，让标签看起来悬浮 */
:deep(.el-tabs__header) {
  margin: 0;
  border-bottom: none !important;
}

/* 2. 导航容器样式调整 */
:deep(.el-tabs__nav) {
  border: none !important;
  border-radius: 0 !important;
}

:deep(.el-tabs__nav-wrap) {
  margin-bottom: 0;
  display: flex;
  align-items: center;
  height: 44px;
}

/* 3. 标签项基础样式 */
:deep(.el-tabs__item) {
  height: 34px !important;
  line-height: 34px !important;
  border: 1px solid #e5e7eb !important; /* gray-200 */
  border-radius: 4px !important;
  margin-right: 6px !important;
  color: #6b7280 !important; /* gray-500 */
  font-size: 13px;
  font-weight: 400;
  padding: 0 16px !important;
  transition: all 0.2s;
  background-color: #fff;
}

/* 4. 标签项悬停样式 */
:deep(.el-tabs__item:hover) {
  color: var(--el-color-primary) !important;
  background-color: #eff6ff; /* sky-50 */
  border-color: #bfdbfe !important; /* sky-200 */
}

/* 5. 标签项激活样式 */
:deep(.el-tabs__item.is-active) {
  color: #fff !important;
  background-color: var(--el-color-primary) !important;
  border-color: var(--el-color-primary) !important;
  font-weight: 500;
}

/* 6. 去除激活标签左侧的圆点装饰（更简洁） */
:deep(.el-tabs__item.is-active::before) {
  display: none;
}

/* 7. 关闭按钮样式微调 */
:deep(.el-tabs__item .is-icon-close) {
  width: 14px;
  margin-left: 6px;
  margin-right: -4px; /* 调整位置 */
  vertical-align: middle;
  color: inherit; /* 跟随文字颜色 */
  transition: all 0.2s;
}

:deep(.el-tabs__item .is-icon-close:hover) {
  background-color: rgba(0, 0, 0, 0.1);
  color: inherit;
}

/* 激活状态下的关闭按钮 hover */
:deep(.el-tabs__item.is-active .is-icon-close:hover) {
  background-color: rgba(255, 255, 255, 0.2);
}

/* 8. 左右滚动箭头样式 */
:deep(.el-tabs__nav-prev), :deep(.el-tabs__nav-next) {
  line-height: 44px;
  font-size: 16px;
  color: #9ca3af;
  padding: 0 4px;
}

:deep(.el-tabs__nav-prev:hover), :deep(.el-tabs__nav-next:hover) {
  color: var(--el-color-primary);
}
</style>