<script setup>
import AdminFooter from "@/layouts/admin/components/AdminFooter.vue";
import AdminHeader from "@/layouts/admin/components/AdminHeader.vue";
import AdminMenu from "@/layouts/admin/components/AdminMenu.vue";
import AdminTagList from "@/layouts/admin/components/AdminTagList.vue";

import {useMenuStore} from "@/stores/menu.js";
import {onMounted} from 'vue';

const menuStore = useMenuStore();

onMounted(() => {
  // 移除 html 标签中的 class="dark"，确保后台管理系统强制为浅色模式
  document.documentElement.classList.remove('dark');
})
</script>

<template>
  <!-- 外层容器：全屏，背景色淡灰，提升质感 -->
  <el-container class="min-h-screen w-full bg-gray-50/50 font-sans">

    <!-- 左侧导航栏 -->
    <!-- transition-[width] 确保宽度变化时的动画流畅 -->
    <el-aside :width="menuStore.menuWidth"
              class="transition-[width] duration-300 ease-in-out shrink-0 relative z-50 h-screen overflow-hidden box-border">
      <AdminMenu/>
    </el-aside>

    <!-- 右侧内容区 -->
    <el-container class="flex flex-col min-w-0 flex-1 transition-all duration-300 h-screen">

      <!-- 顶栏容器 -->
      <!-- 移除默认 padding，添加底部阴影和背景色 -->
      <el-header class="h-[64px] p-0 z-40 bg-white shadow-sm sticky top-0 border-b border-gray-100">
        <AdminHeader/>
      </el-header>

      <!-- 标签页导航 (AdminTagList 内部使用 fixed 定位和占位符，这里直接引入即可) -->
      <AdminTagList/>

      <!-- 主内容区域 -->
      <!-- overflow-x-hidden 防止横向滚动条，relative 用于定位 -->
      <el-main class="flex-grow p-5 overflow-x-hidden overflow-y-auto relative scroll-smooth custom-scrollbar">
        <!-- 根据路由动态展示不同页面 -->
        <router-view v-slot="{ Component }">
          <Transition mode="out-in" name="fade-slide">
            <!-- 包裹 div 确保动画期间布局稳定，key 确保组件正确重绘 -->
            <div :key="$route.path" class="w-full min-h-[calc(100vh-180px)]">
              <KeepAlive :max="10">
                <component :is="Component"></component>
              </KeepAlive>
            </div>
          </Transition>
        </router-view>
      </el-main>

      <!-- 底栏容器 -->
      <el-footer class="h-auto p-0 flex-shrink-0 bg-white border-t border-gray-200">
        <AdminFooter/>
      </el-footer>
    </el-container>
  </el-container>
</template>

<style scoped>
/* 覆盖 Element Plus 默认样式 */
.el-header, .el-footer {
  padding: 0 !important;
  height: auto;
}

/* 自定义滚动条样式 (Chrome/Safari/Edge) */
.custom-scrollbar::-webkit-scrollbar {
  width: 6px;
  height: 6px;
}

.custom-scrollbar::-webkit-scrollbar-track {
  background: transparent;
}

.custom-scrollbar::-webkit-scrollbar-thumb {
  background: #cbd5e1;
  border-radius: 4px;
}

.custom-scrollbar::-webkit-scrollbar-thumb:hover {
  background: #94a3b8;
}

/* 内容区域过渡动画：淡入淡出 + 轻微 Y 轴位移，更具现代感 */
.fade-slide-enter-from {
  opacity: 0;
  transform: translateY(15px);
}

.fade-slide-enter-active,
.fade-slide-leave-active {
  transition: all 0.3s cubic-bezier(0.55, 0, 0.1, 1);
}

.fade-slide-enter-to {
  opacity: 1;
  transform: translateY(0);
}

.fade-slide-leave-from {
  opacity: 1;
  transform: translateY(0);
}

.fade-slide-leave-to {
  opacity: 0;
  transform: translateY(-15px);
}
</style>