<script setup>
import {onMounted, ref} from 'vue'
import {getUserList, updateUserStatus} from "@/api/admin/user.js";
import {showMessage} from '@/composables/utils'
import {Clock} from '@element-plus/icons-vue'

// 表格加载 Loading
const tableLoading = ref(false)
// 表格数据
const tableData = ref([])

function getAllUsers() {
  tableLoading.value = true
  getUserList().then(res => {
    tableData.value = res.data
  }).finally(() => {
    tableLoading.value = false
  })
}

// 更改用户状态
// 适配 el-switch 的 change 事件
const handleStatusChange = (user, newVal) => {
  // 记录旧值，以便失败时回滚
  const oldVal = newVal === 1 ? 0 : 1

  updateUserStatus(user.id, newVal).then(res => {
    if (res.success) {
      showMessage(`用户已${newVal === 1 ? '启用' : '禁用'}`)
    } else {
      // 失败，回滚状态
      user.status = oldVal
      showMessage(res.message || `操作失败`, 'error')
    }
  }).catch(() => {
    // 异常，回滚状态
    user.status = oldVal
    showMessage('网络异常，操作失败', 'error')
  })
}

// 初始化加载数据
onMounted(() => {
  getAllUsers()
})
</script>

<template>
  <div class="p-6">
    <!-- 顶栏：标题和刷新按钮 -->
    <div class="flex items-center justify-between mb-6">
      <div>
        <h2 class="text-xl font-bold text-gray-800">用户管理</h2>
        <p class="text-sm text-gray-500 mt-1">查看系统注册用户，管理账号状态及权限</p>
      </div>
      <el-button :loading="tableLoading" circle icon="Refresh" @click="getAllUsers"></el-button>
    </div>

    <!-- 表格区域 -->
    <el-card class="border-0 !bg-white !rounded-xl table-wrapper" shadow="never">
      <el-table
          v-loading="tableLoading"
          :data="tableData"
          :header-cell-style="{ background: '#f8fafc', color: '#64748b', fontWeight: '600' }"
          highlight-current-row
          style="width: 100%"
      >
        <!-- 用户信息列 (整合 ID, 头像, 用户名) -->
        <el-table-column label="用户" min-width="220">
          <template #default="scope">
            <div class="flex items-center gap-3">
              <!-- 头像 -->
              <el-avatar
                  :size="40"
                  :src="scope.row.avatar"
                  class="border border-gray-200 flex-shrink-0"
              >
                {{ scope.row.username ? scope.row.username.charAt(0).toUpperCase() : 'U' }}
              </el-avatar>
              <!-- 文字信息 -->
              <div class="flex flex-col">
                <span class="font-medium text-gray-800">{{ scope.row.username }}</span>
                <span class="text-xs text-gray-400 font-mono">ID: {{ scope.row.id }}</span>
              </div>
            </div>
          </template>
        </el-table-column>

        <el-table-column label="昵称" min-width="150" prop="nickname">
          <template #default="scope">
            {{ scope.row.nickname || '-' }}
          </template>
        </el-table-column>

        <el-table-column label="注册时间" prop="createTime" sortable width="200">
          <template #default="scope">
            <div class="flex items-center gap-1.5 text-gray-500 text-sm">
              <el-icon>
                <Clock/>
              </el-icon>
              <span>{{ scope.row.createTime }}</span>
            </div>
          </template>
        </el-table-column>

        <!-- 状态列 (整合操作) -->
        <el-table-column label="状态" width="150">
          <template #default="scope">
            <div class="flex items-center gap-2">
              <el-switch
                  v-model="scope.row.status"
                  :active-value="1"
                  :inactive-value="0"
                  active-text="正常"
                  inactive-text="禁用"
                  inline-prompt
                  style="--el-switch-on-color: #10b981; --el-switch-off-color: #ef4444;"
                  @change="(val) => handleStatusChange(scope.row, val)"
              />
            </div>
          </template>
        </el-table-column>

        <!-- 预留操作列 (虽然目前没有其他操作，但为了保持表格结构完整性，可以保留或暂时移除) -->
        <!-- 如果后续有编辑用户、分配角色等功能，可以在这里添加 -->
        <!-- <el-table-column label="操作" width="120" fixed="right">
            <template #default="scope">
                <el-button type="primary" link icon="Edit" disabled>编辑</el-button>
            </template>
        </el-table-column> -->
      </el-table>
    </el-card>
  </div>
</template>

<style scoped>
/* 覆盖表格悬停样式 */
:deep(.el-table--enable-row-hover .el-table__body tr:hover > td.el-table__cell) {
  background-color: #f8fafc;
}

/* 调整 Switch 内部文字大小，防止过长被遮挡 */
:deep(.el-switch__core .el-switch__inner) {
  font-size: 12px;
}
</style>