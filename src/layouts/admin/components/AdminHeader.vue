<template>
  <el-affix :offset="0">
    <div
        class="bg-white/95 backdrop-blur-sm h-[64px] flex items-center px-4 shadow-sm border-b border-gray-100 dark:bg-gray-900/90 dark:border-gray-800 transition-all duration-300">

      <!-- 左侧：折叠按钮 + 面包屑 -->
      <div class="flex items-center gap-4">
        <!-- 折叠/展开按钮 -->
        <div
            class="w-8 h-8 flex items-center justify-center rounded-lg cursor-pointer text-gray-500 hover:bg-gray-100 hover:text-sky-600 transition-all duration-200"
            @click="handleMenuWidth">
          <el-icon size="20">
            <Fold v-if="menuStore.menuWidth === '250px'"/>
            <Expand v-else/>
          </el-icon>
        </div>

        <!-- 面包屑导航 -->
        <el-breadcrumb :separator-icon="ArrowRight" class="hidden md:flex">
          <el-breadcrumb-item :to="{ path: '/admin/index' }">首页</el-breadcrumb-item>
          <template v-for="(item, index) in breadcrumbs" :key="index">
            <el-breadcrumb-item v-if="item.path !== '/admin/index'">{{ item.title }}</el-breadcrumb-item>
          </template>
        </el-breadcrumb>
      </div>

      <!-- 右侧：功能区 -->
      <div class="ml-auto flex items-center gap-2">

        <!-- 跳转前台 -->
        <el-tooltip content="跳转前台" effect="dark" placement="bottom">
          <div
              class="w-9 h-9 flex items-center justify-center rounded-full cursor-pointer text-gray-500 hover:bg-gray-100 hover:text-sky-600 transition-all duration-200"
              @click="router.push('/')">
            <el-icon size="18">
              <Monitor/>
            </el-icon>
          </div>
        </el-tooltip>

        <!-- 刷新 -->
        <el-tooltip content="刷新页面" effect="dark" placement="bottom">
          <div
              class="w-9 h-9 flex items-center justify-center rounded-full cursor-pointer text-gray-500 hover:bg-gray-100 hover:text-green-600 transition-all duration-200"
              @click="handleRefresh">
            <el-icon size="18">
              <Refresh/>
            </el-icon>
          </div>
        </el-tooltip>

        <!-- 全屏 -->
        <el-tooltip content="全屏模式" effect="dark" placement="bottom">
          <div
              class="w-9 h-9 flex items-center justify-center rounded-full cursor-pointer text-gray-500 hover:bg-gray-100 hover:text-orange-500 transition-all duration-200 mr-2"
              @click="toggle">
            <el-icon size="18">
              <FullScreen v-if="!isFullscreen"/>
              <Aim v-else/>
            </el-icon>
          </div>
        </el-tooltip>

        <!-- 分隔线 -->
        <div class="h-6 w-px bg-gray-200 mx-2"></div>

        <!-- 用户下拉菜单 -->
        <el-dropdown trigger="click" @command="handleCommand">
          <div
              class="flex items-center gap-2 cursor-pointer py-1 px-2 rounded-full hover:bg-gray-50 transition-colors duration-200 outline-none">
            <el-avatar :size="32" :src="userInfo.avatar" class="border border-gray-200"/>
            <div class="flex flex-col items-start">
              <span class="text-sm font-medium text-gray-700 dark:text-gray-200 leading-tight">{{
                  userStore.userInfo.username || 'Admin'
                }}</span>
            </div>
            <el-icon class="text-gray-400">
              <ArrowDown/>
            </el-icon>
          </div>

          <template #dropdown>
            <el-dropdown-menu class="min-w-[140px]">
              <div class="px-4 py-2 border-b border-gray-100 mb-1">
                <p class="text-xs text-gray-400">当前用户</p>
                <p class="text-sm font-medium text-gray-800 truncate">{{ userStore.userInfo.username }}</p>
              </div>
              <el-dropdown-item command="updatePassword" icon="Lock">修改密码</el-dropdown-item>
              <el-dropdown-item class="text-red-500 hover:!text-red-600 hover:!bg-red-50" command="logout" divided
                                icon="SwitchButton">退出登录
              </el-dropdown-item>
            </el-dropdown-menu>
          </template>
        </el-dropdown>

      </div>
    </div>
  </el-affix>

  <!-- 修改密码弹窗 (保持原有逻辑) -->
  <FormDialog ref="formDialogRef" destroyOnClose title="修改密码" width="400px" @submit="onSubmit">
    <el-form ref="formRef" :model="form" :rules="rules" label-position="top" size="large">
      <el-form-item label="用户名" prop="username">
        <el-input v-model="form.username" disabled prefix-icon="User"/>
      </el-form-item>
      <el-form-item label="新密码" prop="password">
        <el-input v-model="form.password" placeholder="请输入新密码" prefix-icon="Lock" show-password type="password"/>
      </el-form-item>
      <el-form-item label="确认密码" prop="rePassword">
        <el-input v-model="form.rePassword" placeholder="请再次输入新密码" prefix-icon="Lock" show-password
                  type="password"/>
      </el-form-item>
    </el-form>
  </FormDialog>
</template>

<script setup>
import {Aim, ArrowDown, ArrowRight, Expand, Fold, FullScreen, Monitor, Refresh} from "@element-plus/icons-vue";
import {useMenuStore} from '@/stores/menu.js'
import {computed, reactive, ref, watch} from "vue";
import FormDialog from "@/components/FormDialog.vue";
import {useFullscreen} from '@vueuse/core'
import {useUserStore} from '@/stores/user'
import {useRoute, useRouter} from "vue-router";
import {showMessage, showModel} from "@/composables/utils.js";
import {getUserInfo, updateAdminPassword} from "@/api/admin/user.js";

const menuStore = useMenuStore()
const {isFullscreen, toggle} = useFullscreen()
const userStore = useUserStore()
const router = useRouter()
const route = useRoute()

// 面包屑数据
const breadcrumbs = computed(() => {
  return route.matched.filter(item => item.meta && item.meta.title).map(item => ({
    title: item.meta.title,
    path: item.path
  }))
})

const handleMenuWidth = () => {
  menuStore.handleMenuWidth()
}

const formDialogRef = ref(null)

const handleCommand = (command) => {
  if (command === 'updatePassword') {
    formDialogRef.value.open()
  } else if (command === 'logout') {
    logout()
  }
}

function logout() {
  showModel('是否确认要退出登录？').then(() => {
    userStore.logout()
    showMessage('退出登录成功！')
    router.push('/login')
  })
}

const formRef = ref(null)

const form = reactive({
  username: userStore.userInfo.username || '',
  password: '',
  rePassword: ''
})

const onSubmit = () => {
  formRef.value.validate((valid) => {
    if (!valid) return false

    if (form.password !== form.rePassword) {
      showMessage('两次密码输入不一致，请检查！', 'warning')
      return
    }

    formDialogRef.value.showBtnLoading()
    updateAdminPassword(form).then((res) => {
      if (res.success === true) {
        showMessage('密码重置成功，请重新登录！')
        userStore.logout()
        formDialogRef.value.close()
        router.push('/login')
      } else {
        showMessage(res.message, 'error')
      }
    }).finally(() => formDialogRef.value.closeBtnLoading())
  })
}

const rules = {
  username: [{required: true, message: '用户名不能为空', trigger: 'blur'}],
  password: [{required: true, message: '密码不能为空', trigger: 'blur'}],
  rePassword: [{required: true, message: '确认密码不能为空', trigger: 'blur'}]
}

const handleRefresh = () => {
  location.reload()
}

const userInfo = ref({})
getUserInfo().then(res => {
  userInfo.value = res.data
})

watch(() => userStore.userInfo.username, (newValue) => {
  form.username = newValue
});
</script>

<style scoped>
/* 覆盖 Element Plus 面包屑样式 */
:deep(.el-breadcrumb__inner) {
  font-weight: normal;
  color: #64748b;
}

:deep(.el-breadcrumb__inner.is-link:hover) {
  color: var(--el-color-primary);
}

:deep(.el-breadcrumb__item:last-child .el-breadcrumb__inner) {
  color: #1e293b;
  font-weight: 600;
}
</style>