<template>
  <div class="p-6">
    <!-- 顶栏：标题和添加按钮 -->
    <div class="flex items-center justify-between mb-6">
      <div>
        <h2 class="text-xl font-bold text-gray-800">角色管理</h2>
        <p class="text-sm text-gray-500 mt-1">配置系统角色，分配菜单权限及管理角色下的用户</p>
      </div>
      <el-button icon="Plus" size="large" type="primary" @click="addRole">添加角色</el-button>
    </div>

    <!-- 表格区域 -->
    <el-card class="border-0 !bg-white !rounded-xl table-wrapper" shadow="never">
      <el-table
          v-loading="tableLoading"
          :data="roleTableData"
          :header-cell-style="{ background: '#f8fafc', color: '#64748b', fontWeight: '600' }"
          highlight-current-row
          style="width: 100%"
      >
        <el-table-column label="角色名称" min-width="150" prop="roleName">
          <template #default="scope">
            <div class="flex items-center gap-2">
              <div class="w-8 h-8 rounded-lg bg-indigo-50 text-indigo-600 flex items-center justify-center">
                <el-icon>
                  <user-filled/>
                </el-icon>
              </div>
              <span class="font-medium text-gray-700">{{ scope.row.roleName }}</span>
            </div>
          </template>
        </el-table-column>

        <el-table-column label="角色代码" prop="roleCode" width="180">
          <template #default="scope">
            <el-tag class="font-mono" effect="plain" type="info">{{ scope.row.roleCode }}</el-tag>
          </template>
        </el-table-column>

        <el-table-column label="描述" min-width="200" prop="description">
          <template #default="scope">
            <span :title="scope.row.description"
                  class="text-gray-500 text-sm truncate block">{{ scope.row.description || '-' }}</span>
          </template>
        </el-table-column>

        <el-table-column fixed="right" label="操作" width="280">
          <template #default="scope">
            <div class="flex items-center gap-3">
              <el-button icon="User" link type="primary" @click="viewRoleUsers(scope.row)">
                用户
              </el-button>
              <el-button icon="Key" link type="warning" @click="manageRolePermissions(scope.row)">
                权限
              </el-button>
              <el-button icon="Edit" link type="primary" @click="editRole(scope.row)">
                编辑
              </el-button>
              <!-- 预留删除按钮，虽原逻辑未实现，但UI上留位置 -->
              <!-- <el-button type="danger" link icon="Delete">删除</el-button> -->
            </div>
          </template>
        </el-table-column>
      </el-table>
    </el-card>

    <!-- 角色用户管理对话框 -->
    <el-dialog
        v-model="roleUsersDialogVisible"
        :title="getRoleDialogTitle"
        class="custom-dialog"
        width="800px"
        @close="resetPermissionData"
    >
      <div class="px-2 pb-4">
        <el-alert :closable="false" class="mb-4" show-icon type="info">
          <template #default>
            当前角色下的用户列表，您可以将用户迁移至其他角色。
          </template>
        </el-alert>
        <el-table :data="roleUsers" border max-height="400" stripe style="width: 100%">
          <el-table-column align="center" label="ID" prop="userId" width="80"/>
          <el-table-column label="用户名" min-width="150" prop="username">
            <template #default="scope">
              <div class="font-medium text-gray-700">{{ scope.row.username }}</div>
            </template>
          </el-table-column>
          <el-table-column label="昵称" min-width="150" prop="nickname"/>
          <el-table-column align="center" label="操作" width="120">
            <template #default="scope">
              <el-button plain size="small" type="primary" @click="changeUserRole(scope.row)">移交角色</el-button>
            </template>
          </el-table-column>
        </el-table>
      </div>
    </el-dialog>

    <!-- 角色权限管理对话框 -->
    <el-dialog
        v-model="rolePermissionDialogVisible"
        :title="getPermissionDialogTitle"
        class="custom-dialog"
        top="5vh"
        width="900px"
        @close="resetPermissionData"
    >
      <div class="h-[60vh] flex flex-col">
        <div class="flex items-center gap-2 mb-4 px-1">
          <span class="text-sm text-gray-500">当前正在编辑角色：</span>
          <el-tag effect="dark" type="warning">{{ currentRole.roleName }}</el-tag>
        </div>

        <div class="flex-1 overflow-hidden border border-gray-200 rounded-lg flex">
          <!-- 左侧 Tabs 导航 -->
          <el-tabs
              v-model="activePermissionTab"
              class="h-full custom-tabs w-full"
              tab-position="left"
          >
            <el-tab-pane
                v-for="(permissionGroup, index) in permissionGroups"
                :key="index"
                :name="permissionGroup.permissionParentName"
            >
              <template #label>
                <div class="w-32 text-left truncate flex items-center justify-between pr-2">
                  <span>{{ permissionGroup.permissionParentName }}</span>
                  <el-tag round size="small" type="info">{{
                      permissionGroup.permissionList.filter(p => p.status).length
                    }}
                  </el-tag>
                </div>
              </template>

              <!-- 右侧权限选择区域 -->
              <div class="h-full overflow-y-auto p-4 bg-gray-50/50">
                <div class="grid grid-cols-1 md:grid-cols-2 gap-3">
                  <div
                      v-for="permission in permissionGroup.permissionList"
                      :key="permission.permissionId"
                      :class="permission.status ? 'bg-indigo-50 border-indigo-200' : 'bg-white border-gray-200 hover:border-indigo-200'"
                      class="relative flex items-start p-3 rounded-lg border transition-all cursor-pointer hover:shadow-sm"
                      @click="permission.status = !permission.status"
                  >
                    <div class="flex items-center h-5">
                      <el-checkbox v-model="permission.status" @click.stop/>
                    </div>
                    <div class="ml-3 text-sm">
                      <label :class="{'text-indigo-700': permission.status}"
                             class="font-medium text-gray-700 cursor-pointer">
                        {{ permission.permissionName }}
                      </label>
                      <p class="text-gray-400 text-xs mt-0.5">ID: {{ permission.permissionId }}</p>
                    </div>
                  </div>
                </div>
              </div>
            </el-tab-pane>
          </el-tabs>
        </div>
      </div>

      <template #footer>
        <div class="flex justify-between items-center w-full px-2">
          <span class="text-xs text-gray-400">请谨慎操作权限变更</span>
          <div>
            <el-button @click="rolePermissionDialogVisible = false">取消</el-button>
            <el-button :loading="permissionSaving" type="primary" @click="saveRolePermissions">保存配置</el-button>
          </div>
        </div>
      </template>
    </el-dialog>

    <!-- 更改用户角色对话框 -->
    <el-dialog v-model="changeUserRoleDialogVisible" class="custom-dialog" title="更改用户角色" width="450px">
      <el-form :model="changeRoleForm" label-position="top">
        <el-form-item label="当前用户">
          <div class="w-full p-3 bg-gray-50 rounded border border-gray-200 text-gray-700 font-medium">
            {{ selectedUser.username }}
          </div>
        </el-form-item>
        <el-form-item label="原角色">
          <el-tag type="info">{{ currentRole.roleName }}</el-tag>
        </el-form-item>
        <el-form-item label="变更至新角色">
          <el-select v-model="changeRoleForm.newRoleId" class="w-full" placeholder="请选择目标角色" size="large">
            <el-option
                v-for="role in roleTableData"
                :key="role.id"
                :disabled="role.id === currentRole.id"
                :label="role.roleName"
                :value="role.id"
            />
          </el-select>
        </el-form-item>
      </el-form>
      <template #footer>
        <span class="dialog-footer">
          <el-button @click="changeUserRoleDialogVisible = false">取消</el-button>
          <el-button :disabled="!changeRoleForm.newRoleId" type="primary"
                     @click="confirmChangeUserRole">确认变更</el-button>
        </span>
      </template>
    </el-dialog>

    <!-- 添加/编辑角色对话框 -->
    <el-dialog v-model="addRoleDialogVisible" :title="editingRole ? '编辑角色' : '添加角色'" class="custom-dialog"
               width="500px">
      <el-form ref="roleFormRef" :model="roleForm" :rules="roleFormRules" label-position="top" size="large">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="roleForm.roleName" placeholder="例如：内容管理员" prefix-icon="UserFilled"/>
        </el-form-item>
        <el-form-item label="角色代码" prop="roleCode">
          <el-input v-model="roleForm.roleCode" :disabled="editingRole" placeholder="例如：ROLE_EDITOR"
                    prefix-icon="Postcard"/>
          <span class="text-xs text-gray-400 mt-1">建议使用 ROLE_ 前缀，创建后不可修改（逻辑需后端支持）</span>
        </el-form-item>
        <el-form-item label="描述说明" prop="description">
          <el-input v-model="roleForm.description" :rows="3" placeholder="请描述该角色的职责范围..." type="textarea"/>
        </el-form-item>
      </el-form>
      <template #footer>
        <span class="dialog-footer">
          <el-button @click="addRoleDialogVisible = false">取消</el-button>
          <el-button type="primary" @click="saveRole">保存</el-button>
        </span>
      </template>
    </el-dialog>
  </div>
</template>

<script setup>
import {computed, onMounted, reactive, ref} from 'vue'
import {UserFilled} from '@element-plus/icons-vue'
import {getUserList, updateUserRole} from '@/api/admin/user.js'
import {getRole, getRoleUserList, updateRolePermissions} from '@/api/admin/role.js'
import {getRolePermissionList} from '@/api/admin/permission.js'
import {showMessage} from '@/composables/utils'

// 表格加载 Loading
const tableLoading = ref(false)
// 角色表格数据
const roleTableData = ref([])

// 角色用户管理对话框显示控制
const roleUsersDialogVisible = ref(false)
// 角色权限管理对话框显示控制
const rolePermissionDialogVisible = ref(false)
// 添加/编辑角色对话框显示控制
const addRoleDialogVisible = ref(false)

// 当前查看的角色
const currentRole = ref({})

// 角色用户数据
const roleUsers = ref([])

// 计算属性: 用户对话框标题
const getRoleDialogTitle = computed(() => {
  const roleName = currentRole.value?.roleName || '未知角色'
  return `角色用户管理 - ${roleName}`
})

// 计算属性: 权限对话框标题
const getPermissionDialogTitle = computed(() => {
  return `权限配置`
})

// 更改用户角色对话框显示控制
const changeUserRoleDialogVisible = ref(false)
// 选中的用户
const selectedUser = ref({})

// 更改角色表单
const changeRoleForm = reactive({
  newRoleId: ''
})

// 所有用户
const allUsers = ref([])

// 权限标签页
const activePermissionTab = ref('')

// 权限组数据
const permissionGroups = ref([])

// 是否正在保存权限
const permissionSaving = ref(false)

// 是否正在编辑角色
const editingRole = ref(false)

// 角色表单
const roleForm = reactive({
  roleCode: '',
  roleName: '',
  description: ''
})

// 角色表单验证规则
const roleFormRules = {
  roleCode: [
    {required: true, message: '请输入角色代码', trigger: 'blur'}
  ],
  roleName: [
    {required: true, message: '请输入角色名称', trigger: 'blur'}
  ],
  description: [
    {required: true, message: '请输入角色描述', trigger: 'blur'}
  ]
}

// 表单引用
const roleFormRef = ref()

// 角色列表
const roleList = ref([])

// 获取所有用户
function getAllUsers() {
  // tableLoading.value = true // 不应影响主表格loading
  getUserList().then(res => {
    // tableLoading.value = false
    allUsers.value = res.data
  })
}

// 获取角色列表
function getRoleList() {
  tableLoading.value = true
  getRole().then(res => {
    roleList.value = res.data
    roleTableData.value = res.data
  }).finally(() => {
    tableLoading.value = false
  })
}

// 查看角色用户
const viewRoleUsers = (role) => {
  currentRole.value = role
  // 调用接口获取角色下用户
  const loading = true // 局部loading逻辑可优化
  getRoleUserList(role.id).then(res => {
    roleUsers.value = res.data
    roleUsersDialogVisible.value = true
  }).catch(() => {
    showMessage('获取用户列表失败', 'error')
  })
}

// 管理角色权限
const manageRolePermissions = (role) => {
  currentRole.value = role

  // 加载所有权限
  // tableLoading.value = true // 避免主表格闪烁
  const loading = true
  getRolePermissionList(role.id).then((res) => {
    // 设置所有权限
    permissionGroups.value = res.data

    // 设置当前角色的权限状态
    permissionGroups.value.forEach(permissionGroup => {
      permissionGroup.permissionList.forEach(permission => {
        // 使用响应式方式设置状态
        permission.status = Boolean(permission.status) // 确保是布尔值
      })
    })

    // 设置默认标签页为第一个
    if (permissionGroups.value.length > 0) {
      activePermissionTab.value = permissionGroups.value[0].permissionParentName
    }

    rolePermissionDialogVisible.value = true
  })
      .catch(() => {
        showMessage('加载权限数据失败', 'error')
      })
}

// 重置权限数据
const resetPermissionData = () => {
  permissionGroups.value = []
  activePermissionTab.value = ''
}

// 保存角色权限
const saveRolePermissions = () => {
  // 收集选中的权限ID
  const selectedPermissionIds = []
  permissionGroups.value.forEach(permissionGroup => {
    permissionGroup.permissionList.forEach(permission => {
      if (permission.status) {
        selectedPermissionIds.push(permission.permissionId)
      }
    })
  })

  // 保存权限操作
  permissionSaving.value = true
  updateRolePermissions(currentRole.value.id, selectedPermissionIds)
      .then(() => {
        showMessage(`角色"${currentRole.value.roleName}"权限保存成功`, 'success')
        rolePermissionDialogVisible.value = false
      })
      .catch(() => {
        showMessage('权限保存失败', 'error')
      })
      .finally(() => {
        permissionSaving.value = false
      })
}

// 编辑角色
const editRole = (role) => {
  editingRole.value = true
  roleForm.roleCode = role.roleCode
  roleForm.roleName = role.roleName
  roleForm.description = role.description
  currentRole.value = role
  addRoleDialogVisible.value = true
}

// 添加角色
const addRole = () => {
  editingRole.value = false
  roleForm.roleCode = ''
  roleForm.roleName = ''
  roleForm.description = ''
  addRoleDialogVisible.value = true
}

// 保存角色
const saveRole = () => {
  roleFormRef.value.validate((valid) => {
    if (valid) {
      if (editingRole.value) {
        // 模拟编辑角色操作
        showMessage(`角色"${roleForm.roleName}"编辑成功`, 'success')
      } else {
        // 模拟添加角色操作
        showMessage(`角色"${roleForm.roleName}"添加成功`, 'success')
      }
      addRoleDialogVisible.value = false
      // 重新加载角色列表
      getRoleList()
    }
  })
}

// 更改用户角色
const changeUserRole = (user) => {
  selectedUser.value = user
  changeRoleForm.newRoleId = ''
  changeUserRoleDialogVisible.value = true
}

// 确认更改用户角色
const confirmChangeUserRole = () => {
  if (!changeRoleForm.newRoleId) {
    showMessage('请选择新角色', 'error')
    return
  }

  // 更改用户角色操作
  updateUserRole(selectedUser.value.username, changeRoleForm.newRoleId).then((res) => {
    showMessage(`用户"${selectedUser.value.username}"已更改到新角色`, 'success')
    // 重新加载当前角色用户
    viewRoleUsers(currentRole.value)
  })
      .catch(() => {
        showMessage('用户角色更改失败', 'error')
      })

  changeUserRoleDialogVisible.value = false
}

// 初始化加载数据
onMounted(() => {
  // getAllUsers() // 暂时没用到，可注释
  getRoleList()
})
</script>

<style scoped>
/* 覆盖表格悬停样式 */
:deep(.el-table--enable-row-hover .el-table__body tr:hover > td.el-table__cell) {
  background-color: #f8fafc;
}

/* 侧边 Tabs 样式定制 */
:deep(.custom-tabs .el-tabs__header.is-left) {
  margin-right: 0;
  border-right: 1px solid #e5e7eb;
  background-color: white;
  width: 140px;
}

:deep(.custom-tabs .el-tabs__item) {
  text-align: left;
  justify-content: flex-start;
  padding: 0 16px !important;
  height: 50px;
  line-height: 50px;
}

:deep(.custom-tabs .el-tabs__item.is-active) {
  background-color: #e0e7ff; /* indigo-100 */
  color: #4338ca; /* indigo-700 */
  border-right: 2px solid #4338ca;
}

:deep(.custom-tabs .el-tabs__content) {
  height: 100%;
}

:deep(.custom-tabs .el-tab-pane) {
  height: 100%;
}
</style>