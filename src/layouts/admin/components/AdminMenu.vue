<template>
  <div :style="{ width: menuStore.menuWidth }"
       class="fixed top-0 left-0 bottom-0 bg-slate-900 transition-all duration-300 z-50 flex flex-col shadow-2xl">

    <!-- 顶部 Logo -->
    <div
        :class="isCollapse ? 'px-0' : 'px-4'"
        class="h-[64px] flex items-center justify-center flex-shrink-0 bg-slate-900 border-b border-slate-800 transition-all duration-300 relative overflow-hidden">
      <div class="flex items-center gap-3 w-full justify-center">
        <!-- Logo 图标 -->
        <div
            class="w-8 h-8 rounded-lg bg-white/5 flex items-center justify-center flex-shrink-0 border border-white/10 shadow-sm transition-transform duration-300 hover:scale-105 cursor-pointer"
            @click="router.push('/')">
          <img v-if="blogSettingsStore.blogSettings.logo" :src="blogSettingsStore.blogSettings.logo" alt="Logo"
               class="w-full h-full object-cover rounded-lg"/>
          <span v-else class="text-indigo-400 font-bold text-lg">M</span>
        </div>

        <!-- 标题文字 (展开时显示) -->
        <transition name="fade">
             <span v-show="!isCollapse"
                   class="font-bold text-lg text-gray-100 tracking-wide whitespace-nowrap truncate cursor-default">
                {{ blogSettingsStore.blogSettings.name || 'MeBlog' }}
             </span>
        </transition>
      </div>
    </div>

    <!-- 下方菜单 -->
    <div class="flex-1 py-4 overflow-y-auto overflow-x-hidden custom-scrollbar bg-slate-900">
      <el-menu
          :collapse="isCollapse"
          :collapse-transition="false"
          :default-active="defaultActive"
          :unique-opened="true"
          active-text-color="#fff"
          background-color="transparent"
          class="border-none w-full !border-r-0"
          text-color="#94a3b8"
          @select="handleSelect"
      >
        <template v-for="(item,index) in menus" :key="index">
          <el-menu-item :index="item.path" class="menu-item-custom">
            <el-icon class="menu-icon">
              <component :is="item.icon"></component>
            </el-icon>
            <template #title>
              <span class="font-medium ml-2">{{ item.name }}</span>
            </template>
          </el-menu-item>
        </template>
      </el-menu>
    </div>
  </div>
</template>

<script setup>
import {computed, ref} from "vue";
import {useRoute, useRouter} from "vue-router";
import {useMenuStore} from '@/stores/menu.js'
import {useBlogSettingsStore} from "@/stores/blogsettings.js";

// 引入博客设置信息 store
const blogSettingsStore = useBlogSettingsStore()

const route = useRoute();
const router = useRouter();

// 根据路由地址判断哪个菜单被选中
const defaultActive = ref(route.path)

// 菜单选择事件
const handleSelect = (path) => {
  router.push(path)
}

const menuStore = useMenuStore()

// 是否折叠
const isCollapse = computed(() => !(menuStore.menuWidth === '250px'))

const menus = [
  {
    'name': '仪表盘',
    'icon': 'Monitor',
    'path': '/admin/index'
  },
  {
    'name': '文章管理',
    'icon': 'Document',
    'path': '/admin/article/list',
  },
  {
    'name': '分类管理',
    'icon': 'FolderOpened',
    'path': '/admin/category/list',
  },
  {
    'name': '标签管理',
    'icon': 'PriceTag',
    'path': '/admin/tag/list',
  },
  {
    'name': '知识库管理',
    'icon': 'Collection',
    'path': '/admin/wiki/list',
  },
  {
    'name': '用户管理',
    'icon': 'User',
    'path': '/admin/user',
  },
  {
    'name': '角色管理',
    'icon': 'HomeFilled',
    'path': '/admin/role',
  },
  {
    'name': '文件管理',
    'icon': 'Files',
    'path': '/admin/file',
  },
  {
    'name': '博客设置',
    'icon': 'Setting',
    'path': '/admin/blog/setting',
  },
]
</script>

<style scoped>
/* 淡入淡出动画 */
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.2s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

/* 自定义滚动条 - 适配深色 */
.custom-scrollbar::-webkit-scrollbar {
  width: 4px;
  background: transparent;
}

.custom-scrollbar::-webkit-scrollbar-thumb {
  background: #334155; /* slate-700 */
  border-radius: 4px;
}

.custom-scrollbar::-webkit-scrollbar-thumb:hover {
  background: #475569; /* slate-600 */
}

/* 深度选择器覆盖 Element Plus 样式 */
:deep(.el-menu) {
  border-right: none !important;
  background-color: transparent !important;
}

:deep(.menu-item-custom) {
  margin: 4px 12px !important;
  border-radius: 8px !important;
  height: 50px !important;
  line-height: 50px !important;
  color: #94a3b8 !important; /* slate-400 */
  transition: all 0.3s ease;
  border: 1px solid transparent;
}

:deep(.menu-item-custom:hover) {
  background-color: #1e293b !important; /* slate-800 */
  color: #f8fafc !important; /* slate-50 */
}

/* 选中状态：使用 Indigo 渐变色，更有质感 */
:deep(.el-menu-item.is-active) {
  background: linear-gradient(90deg, #4f46e5 0%, #4338ca 100%) !important; /* indigo-600 to indigo-700 */
  color: #ffffff !important;
  font-weight: 500;
  box-shadow: 0 4px 12px rgba(79, 70, 229, 0.4); /* glow effect */
  border: 1px solid rgba(255, 255, 255, 0.1);
}

/* 选中状态下的图标颜色 */
:deep(.el-menu-item.is-active .el-icon) {
  color: #ffffff !important;
}

/* 非选中状态下图标颜色 */
:deep(.menu-item-custom .el-icon) {
  color: #64748b; /* slate-500 */
  transition: color 0.3s;
}

/* 悬停时图标变亮并微动 */
:deep(.menu-item-custom:hover .el-icon) {
  color: #e2e8f0; /* slate-200 */
  transform: scale(1.1);
}

/* 折叠时的样式微调 */
:deep(.el-menu--collapse .menu-item-custom) {
  margin: 6px 8px !important;
  padding: 0 !important;
  display: flex;
  justify-content: center;
}

:deep(.el-menu--collapse .el-sub-menu__title) {
  padding: 0 !important;
  justify-content: center;
}
</style>