<template>
  <div class="bg-white rounded-xl p-6 mb-6 shadow-sm border border-gray-100 transition-all duration-300 hover:shadow-md">
    <div class="flex flex-wrap items-center justify-between gap-4">
      <!-- 左侧搜索区 -->
      <div class="flex items-center gap-4 flex-wrap flex-1 min-w-0">
        <div class="w-64">
          <slot name="search-input"></slot>
        </div>

        <div class="w-80">
          <slot name="date-picker"></slot>
        </div>

        <div class="flex gap-2">
          <el-button
            icon="Search"
            plain
            type="primary"
            @click="$emit('search')"
            class="transition-all duration-300 hover:shadow-md"
          >
            搜索
          </el-button>
          <el-button
            icon="RefreshRight"
            @click="$emit('reset')"
            class="transition-all duration-300 hover:shadow-sm"
          >
            重置
          </el-button>
        </div>
      </div>

      <!-- 右侧操作区 -->
      <div>
        <slot name="actions"></slot>
      </div>
    </div>
  </div>
</template>

<script setup>
defineEmits(['search', 'reset'])
</script>

<style scoped>
/* 增强悬停效果 */
:deep(.el-button) {
  transition: all 0.3s ease;
}

:deep(.el-button:hover) {
  transform: translateY(-1px);
}
</style>
