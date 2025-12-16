<template>
  <div class="p-4 min-h-screen bg-gray-50 dark:bg-gray-900">
    <!-- 顶部操作栏 -->
    <div class="mb-6 flex flex-col md:flex-row justify-between items-center gap-4 bg-white dark:bg-gray-800 p-4 rounded-lg shadow-sm">
      <h2 class="text-xl font-bold text-gray-700 dark:text-gray-200 flex items-center">
        <span class="mr-2">🏷️</span> 标签云管理
        <span class="ml-2 text-sm font-normal text-gray-500">({{ filteredTags.length }} 个标签)</span>
      </h2>

      <div class="flex gap-3 w-full md:w-auto">
        <el-input
            v-model="searchKeyword"
            placeholder="搜索标签..."
            prefix-icon="Search"
            clearable
            class="w-full md:w-64"
        />
        <el-button type="primary" icon="Plus" @click="openDialog()">新增标签</el-button>
        <el-button icon="Refresh" circle @click="getList" />
      </div>
    </div>

    <!-- 标签云展示区 -->
    <div
        v-loading="loading"
        class="bg-white dark:bg-gray-800 p-8 rounded-xl shadow-sm min-h-[400px]"
    >
      <el-empty v-if="filteredTags.length === 0" description="暂无标签数据" />

      <div v-else class="flex flex-wrap gap-4 content-start">
        <transition-group name="list">
          <div
              v-for="tag in filteredTags"
              :key="tag.id"
              class="group relative cursor-pointer select-none transition-all duration-300 hover:-translate-y-1"
              @click="openDialog(tag)"
          >
            <!-- 标签本体 -->
            <div
                class="px-4 py-2 rounded-full text-sm font-medium text-white shadow-md flex items-center gap-2 transition-transform active:scale-95"
                :style="{ backgroundColor: stringToColor(tag.name) }"
            >
              <span>{{ tag.name }}</span>
              <!-- 如果后端返回了文章关联数，可以在这里显示 -->
              <!-- <span class="bg-white/30 px-1.5 rounded-md text-xs">{{ tag.articleCount || 0 }}</span> -->

              <!-- 删除按钮 (悬停显示或常驻) -->
              <span
                  class="ml-1 p-0.5 rounded-full hover:bg-black/20 text-white opacity-60 hover:opacity-100 transition-opacity"
                  @click.stop="handleDelete(tag)"
              >
                <el-icon><Close /></el-icon>
              </span>
            </div>
          </div>
        </transition-group>
      </div>
    </div>

    <!-- 新增/编辑对话框 -->
    <el-dialog
        v-model="dialogVisible"
        :title="dialogTitle"
        width="400px"
        align-center
        destroy-on-close
    >
      <el-form ref="formRef" :model="form" :rules="rules" label-position="top">
        <el-form-item label="标签名称" prop="name">
          <el-input
              v-model="form.name"
              placeholder="请输入标签名称"
              maxlength="20"
              show-word-limit
              @keyup.enter="submitForm"
          />
        </el-form-item>
      </el-form>
      <template #footer>
        <div class="dialog-footer">
          <el-button @click="dialogVisible = false">取消</el-button>
          <el-button type="primary" :loading="submitLoading" @click="submitForm">
            确定
          </el-button>
        </div>
      </template>
    </el-dialog>
  </div>
</template>

<script setup>
import {computed, onMounted, reactive, ref} from 'vue'
import {addTag, deleteTag, getTagList, updateTag} from '@/api/admin/tag'
import {ElMessage, ElMessageBox} from 'element-plus'
import {Close} from '@element-plus/icons-vue'

// --- 状态定义 ---
const loading = ref(false)
const tags = ref([])
const searchKeyword = ref('')
const dialogVisible = ref(false)
const submitLoading = ref(false)
const formRef = ref(null)

const form = reactive({
  id: Number,
  name: ''
})

const rules = {
  name: [{ required: true, message: '请输入标签名称', trigger: 'blur' }]
}

// --- 计算属性 ---
const filteredTags = computed(() => {
  if (!searchKeyword.value) return tags.value
  return tags.value.filter(tag =>
      tag.name.toLowerCase().includes(searchKeyword.value.toLowerCase())
  )
})

const dialogTitle = computed(() => form.id ? '编辑标签' : '新增标签')

// --- 方法 ---

// 获取列表
const getList = async () => {
  loading.value = true
  try {
    // 这里根据实际 API 调整解构
    const res = await getTagList({})
    // 兼容常见返回结构，若直接返回数组则取res，若在data字段则取res.data
    tags.value = Array.isArray(res) ? res : (res.data?.list || res.data || [])
  } catch (error) {
    console.error('获取标签失败', error)
    ElMessage.error('获取标签列表失败')
  } finally {
    loading.value = false
  }
}

// 打开弹窗
const openDialog = (tag = null) => {
  if (tag) {
    form.id = tag.id
    form.name = tag.name
  } else {
    form.id = null
    form.name = ''
  }
  dialogVisible.value = true
}

// 提交表单
const submitForm = async () => {
  if (!formRef.value) return

  await formRef.value.validate(async (valid) => {
    if (valid) {
      submitLoading.value = true
      try {
        if (form.id) {
          await updateTag(form)
          ElMessage.success('更新成功')
        } else {
          await addTag(form)
          ElMessage.success('添加成功')
        }
        dialogVisible.value = false
        await getList()
      } catch (error) {
        // 错误处理通常由拦截器统一处理，这里可保留作为兜底
      } finally {
        submitLoading.value = false
      }
    }
  })
}

// 删除标签
const handleDelete = (tag) => {
  ElMessageBox.confirm(
      `确定要删除标签 "${tag.name}" 吗？此操作不可恢复。`,
      '警告',
      {
        confirmButtonText: '确定删除',
        cancelButtonText: '取消',
        type: 'warning',
      }
  ).then(async () => {
    try {
      await deleteTag(tag.id)
      ElMessage.success('删除成功')
      // 从本地列表移除，避免重新请求闪烁
      tags.value = tags.value.filter(t => t.id !== tag.id)
    } catch (error) {
      // 错误处理
    }
  })
}

// --- 工具函数 ---

// 根据字符串生成固定的柔和颜色
const stringToColor = (str) => {
  let hash = 0
  for (let i = 0; i < str.length; i++) {
    hash = str.charCodeAt(i) + ((hash << 5) - hash)
  }

  // 使用 HSL 颜色空间生成柔和的颜色
  // Hue: 基于 hash
  // Saturation: 60-80% 之间
  // Lightness: 50-70% 之间，确保文字可读性
  const h = Math.abs(hash) % 360
  const s = 60 + (Math.abs(hash) % 20) // 60% - 80%
  const l = 45 + (Math.abs(hash) % 20) // 45% - 65% (稍深一点以便白字显示)

  return `hsl(${h}, ${s}%, ${l}%)`
}

// --- 生命周期 ---
onMounted(() => {
  getList()
})
</script>

<style scoped>
/* 列表过渡动画 */
.list-move,
.list-enter-active,
.list-leave-active {
  transition: all 0.5s ease;
}

.list-enter-from,
.list-leave-to {
  opacity: 0;
  transform: scale(0.5);
}

/* 确保移除元素时布局平滑变动 */
.list-leave-active {
  position: absolute;
}
</style>