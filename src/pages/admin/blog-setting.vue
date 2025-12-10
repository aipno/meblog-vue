<template>
  <div class="p-6">
    <!-- 顶栏：标题和保存按钮 -->
    <div class="flex items-center justify-between mb-6">
      <div>
        <h2 class="text-xl font-bold text-gray-800">博客设置</h2>
        <p class="text-sm text-gray-500 mt-1">管理您的博客基本信息及个性化配置</p>
      </div>
      <el-button :loading="btnLoading" icon="Check" size="large" type="primary" @click="onSubmit">保存更改</el-button>
    </div>

    <el-form ref="formRef" :model="form" :rules="rules" class="w-full" label-position="top">
      <el-row :gutter="24">
        <!-- 左侧：基础信息 -->
        <el-col :lg="16" :span="24">
          <el-card class="border-0 !bg-white !rounded-xl mb-6" shadow="never">
            <template #header>
              <div class="font-bold text-gray-700 flex items-center gap-2">
                <el-icon>
                  <InfoFilled/>
                </el-icon>
                基础信息
              </div>
            </template>

            <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
              <el-form-item label="博客名称" prop="name">
                <el-input v-model="form.name" clearable placeholder="请输入博客名称" size="large"/>
              </el-form-item>
              <el-form-item label="作者名" prop="author">
                <el-input v-model="form.author" clearable placeholder="请输入作者昵称" size="large"/>
              </el-form-item>
            </div>

            <el-form-item label="介绍语" prop="introduction">
              <el-input
                  v-model="form.introduction"
                  :rows="4"
                  maxlength="200"
                  placeholder="一句话介绍你自己..."
                  show-word-limit
                  type="textarea"
              />
            </el-form-item>

            <el-divider content-position="left">图片设置</el-divider>

            <div class="grid grid-cols-1 md:grid-cols-2 gap-8 mt-6">
              <!-- 博客 LOGO -->
              <el-form-item label="博客 LOGO" prop="logo">
                <div class="w-full flex items-start gap-4">
                  <el-upload
                      :auto-upload="false"
                      :on-change="handleLogoChange"
                      :show-file-list="false"
                      accept="image/*"
                      action="#"
                      class="avatar-uploader-custom"
                  >
                    <div v-if="form.logo"
                         class="relative group w-32 h-32 rounded-xl overflow-hidden border border-gray-200 cursor-pointer shadow-sm">
                      <img :src="form.logo" alt="Logo" class="w-full h-full object-cover"/>
                      <div
                          class="absolute inset-0 bg-black/50 hidden group-hover:flex flex-col items-center justify-center text-white transition-all">
                        <el-icon class="text-2xl mb-1">
                          <Edit/>
                        </el-icon>
                        <span class="text-xs">更换图片</span>
                      </div>
                    </div>
                    <div v-else
                         class="w-32 h-32 rounded-xl border-2 border-dashed border-gray-300 hover:border-sky-500 flex flex-col items-center justify-center text-gray-400 hover:text-sky-500 transition-colors bg-gray-50 cursor-pointer">
                      <el-icon class="text-2xl mb-2">
                        <Plus/>
                      </el-icon>
                      <span class="text-xs">上传 Logo</span>
                    </div>
                  </el-upload>
                  <div class="text-xs text-gray-400 leading-relaxed mt-1">
                    <p>支持 JPG, PNG, GIF 格式</p>
                    <p>建议尺寸：200 x 200 像素</p>
                    <p>将用于网站头部和浏览器标签页</p>
                  </div>
                </div>
              </el-form-item>

              <!-- 作者头像 -->
              <el-form-item label="作者头像" prop="avatar">
                <div class="w-full flex items-start gap-4">
                  <el-upload
                      :auto-upload="false"
                      :on-change="handleAvatarChange"
                      :show-file-list="false"
                      accept="image/*"
                      action="#"
                      class="avatar-uploader-custom"
                  >
                    <div v-if="form.avatar"
                         class="relative group w-32 h-32 rounded-full overflow-hidden border border-gray-200 cursor-pointer shadow-sm">
                      <img :src="form.avatar" alt="Avatar" class="w-full h-full object-cover"/>
                      <div
                          class="absolute inset-0 bg-black/50 hidden group-hover:flex flex-col items-center justify-center text-white transition-all">
                        <el-icon class="text-2xl mb-1">
                          <Edit/>
                        </el-icon>
                        <span class="text-xs">更换头像</span>
                      </div>
                    </div>
                    <div v-else
                         class="w-32 h-32 rounded-full border-2 border-dashed border-gray-300 hover:border-sky-500 flex flex-col items-center justify-center text-gray-400 hover:text-sky-500 transition-colors bg-gray-50 cursor-pointer">
                      <el-icon class="text-2xl mb-2">
                        <Plus/>
                      </el-icon>
                      <span class="text-xs">上传头像</span>
                    </div>
                  </el-upload>
                  <div class="text-xs text-gray-400 leading-relaxed mt-1">
                    <p>支持 JPG, PNG 格式</p>
                    <p>建议使用正方形图片</p>
                    <p>将展示在侧边栏和评论区</p>
                  </div>
                </div>
              </el-form-item>
            </div>
          </el-card>
        </el-col>

        <!-- 右侧：社交信息 -->
        <el-col :lg="8" :span="24">
          <el-card class="border-0 !bg-white !rounded-xl sticky top-6" shadow="never">
            <template #header>
              <div class="font-bold text-gray-700 flex items-center gap-2">
                <el-icon>
                  <Share/>
                </el-icon>
                社交主页
              </div>
            </template>

            <div class="space-y-4">
              <!-- Github -->
              <div
                  class="p-4 bg-gray-50/50 rounded-xl border border-gray-100 hover:shadow-sm hover:border-sky-100 transition-all">
                <div class="flex items-center justify-between mb-2">
                  <div class="flex items-center gap-2">
                    <div class="w-8 h-8 rounded-full bg-gray-800 flex items-center justify-center text-white">
                      <i class="fab fa-github text-lg"></i>
                      <!-- 这里可以用 icon 或 svg，为了简单演示用文字首字母代替，实际项目中建议用 SVG -->
                      <span class="font-bold text-xs">G</span>
                    </div>
                    <span class="text-sm font-semibold text-gray-700">GitHub</span>
                  </div>
                  <el-switch v-model="isGithubChecked" style="--el-switch-on-color: #10b981;"
                             @change="githubSwitchChange"/>
                </div>
                <transition name="el-zoom-in-top">
                  <div v-if="isGithubChecked" class="mt-3">
                    <el-input v-model="form.githubHomepage" clearable placeholder="https://github.com/username">
                      <template #prefix>
                        <el-icon class="text-gray-400">
                          <Link/>
                        </el-icon>
                      </template>
                    </el-input>
                  </div>
                </transition>
              </div>

              <!-- Gitee -->
              <div
                  class="p-4 bg-gray-50/50 rounded-xl border border-gray-100 hover:shadow-sm hover:border-red-100 transition-all">
                <div class="flex items-center justify-between mb-2">
                  <div class="flex items-center gap-2">
                    <div class="w-8 h-8 rounded-full bg-red-600 flex items-center justify-center text-white">
                      <span class="font-bold text-xs">码</span>
                    </div>
                    <span class="text-sm font-semibold text-gray-700">Gitee</span>
                  </div>
                  <el-switch v-model="isGiteeChecked" style="--el-switch-on-color: #10b981;"
                             @change="giteeSwitchChange"/>
                </div>
                <transition name="el-zoom-in-top">
                  <div v-if="isGiteeChecked" class="mt-3">
                    <el-input v-model="form.giteeHomepage" clearable placeholder="https://gitee.com/username">
                      <template #prefix>
                        <el-icon class="text-gray-400">
                          <Link/>
                        </el-icon>
                      </template>
                    </el-input>
                  </div>
                </transition>
              </div>

              <!-- 知乎 -->
              <div
                  class="p-4 bg-gray-50/50 rounded-xl border border-gray-100 hover:shadow-sm hover:border-blue-100 transition-all">
                <div class="flex items-center justify-between mb-2">
                  <div class="flex items-center gap-2">
                    <div class="w-8 h-8 rounded-full bg-blue-500 flex items-center justify-center text-white">
                      <span class="font-bold text-xs">知</span>
                    </div>
                    <span class="text-sm font-semibold text-gray-700">知乎</span>
                  </div>
                  <el-switch v-model="isZhihuChecked" style="--el-switch-on-color: #10b981;"
                             @change="zhihuSwitchChange"/>
                </div>
                <transition name="el-zoom-in-top">
                  <div v-if="isZhihuChecked" class="mt-3">
                    <el-input v-model="form.zhihuHomepage" clearable placeholder="主页链接">
                      <template #prefix>
                        <el-icon class="text-gray-400">
                          <Link/>
                        </el-icon>
                      </template>
                    </el-input>
                  </div>
                </transition>
              </div>

              <!-- CSDN -->
              <div
                  class="p-4 bg-gray-50/50 rounded-xl border border-gray-100 hover:shadow-sm hover:border-orange-100 transition-all">
                <div class="flex items-center justify-between mb-2">
                  <div class="flex items-center gap-2">
                    <div class="w-8 h-8 rounded-full bg-orange-600 flex items-center justify-center text-white">
                      <span class="font-bold text-xs">C</span>
                    </div>
                    <span class="text-sm font-semibold text-gray-700">CSDN</span>
                  </div>
                  <el-switch v-model="isCSDNChecked" style="--el-switch-on-color: #10b981;" @change="csdnSwitchChange"/>
                </div>
                <transition name="el-zoom-in-top">
                  <div v-if="isCSDNChecked" class="mt-3">
                    <el-input v-model="form.csdnHomepage" clearable placeholder="主页链接">
                      <template #prefix>
                        <el-icon class="text-gray-400">
                          <Link/>
                        </el-icon>
                      </template>
                    </el-input>
                  </div>
                </transition>
              </div>
            </div>
          </el-card>
        </el-col>
      </el-row>
    </el-form>
  </div>
</template>

<script setup>
import {reactive, ref} from 'vue'
import {Edit, InfoFilled, Link, Plus, Share} from '@element-plus/icons-vue'
import {getBlogSettingsDetail, updateBlogSettings} from '@/api/admin/blogsettings'
import {uploadFile} from '@/api/admin/file'
import {showMessage} from '@/composables/utils'

// 表单对象
const form = reactive({
  name: '',
  author: '',
  logo: '',
  avatar: '',
  introduction: '',
  githubHomepage: '',
  giteeHomepage: '',
  zhihuHomepage: '',
  csdnHomepage: '',
})

// 是否开启 GitHub
const isGithubChecked = ref(false)
// 是否开启 Gitee
const isGiteeChecked = ref(false)
// 是否开启知乎
const isZhihuChecked = ref(false)
// 是否开启 CSDN
const isCSDNChecked = ref(false)

// 是否显示保存按钮的 loading 状态，默认为 false
const btnLoading = ref(false)

// 表单引用
const formRef = ref(null)

// 规则校验
const rules = {
  name: [{required: true, message: '请输入博客名称', trigger: 'blur'}],
  author: [{required: true, message: '请输入作者名', trigger: 'blur'}],
  logo: [{required: true, message: '请上传博客 LOGO', trigger: 'change'}],
  avatar: [{required: true, message: '请上传作者头像', trigger: 'change'}],
  introduction: [{required: true, message: '请输入介绍语', trigger: 'blur'}],
}

// 监听 Github Switch 改变事件
const githubSwitchChange = (checked) => {
  if (checked === false) {
    form.githubHomepage = ''
  }
}

// 监听 Gitee Switch 改变事件
const giteeSwitchChange = (checked) => {
  if (checked === false) {
    form.giteeHomepage = ''
  }
}

// 监听知乎 Switch 改变事件
const zhihuSwitchChange = (checked) => {
  if (checked === false) {
    form.zhihuHomepage = ''
  }
}

// 监听 CSDN Switch 改变事件
const csdnSwitchChange = (checked) => {
  if (checked === false) {
    form.csdnHomepage = ''
  }
}

// 初始化博客设置数据，并渲染到页面上
function initBlogSettings() {
  // 请求后台接口
  getBlogSettingsDetail().then((e) => {
    if (e.success === true) {
      // 设置表单数据
      form.name = e.data.name
      form.author = e.data.author
      form.logo = e.data.logo
      form.avatar = e.data.avatar
      form.introduction = e.data.introduction

      // 第三方平台信息设置，先判断后端返回平台链接是否为空，若不为空，则将 switch 组件置为选中状态，并设置表单对应数据
      if (e.data.githubHomepage) {
        isGithubChecked.value = true
        form.githubHomepage = e.data.githubHomepage
      }

      if (e.data.giteeHomepage) {
        isGiteeChecked.value = true
        form.giteeHomepage = e.data.giteeHomepage
      }

      if (e.data.zhihuHomepage) {
        isZhihuChecked.value = true
        form.zhihuHomepage = e.data.zhihuHomepage
      }

      if (e.data.csdnHomepage) {
        isCSDNChecked.value = true
        form.csdnHomepage = e.data.csdnHomepage
      }
    }
  })
}

// 上传 logo 图片
const handleLogoChange = (file) => {
  // 表单对象
  let formData = new FormData()
  // 添加 file 字段，并将文件传入
  formData.append('file', file.raw)
  uploadFile(formData).then((e) => {
    // 响参失败，提示错误消息
    if (e.success === false) {
      let message = e.message
      showMessage(message, 'error')
      return
    }

    // 成功则设置 logo 链接，并提示成功
    form.logo = e.data.url
    showMessage('上传成功')
  })
}
// 手动调用一下初始化方法
initBlogSettings()

// 上传作者头像
const handleAvatarChange = (file) => {
  // 表单对象
  let formData = new FormData()
  // 添加 file 字段，并将文件传入
  formData.append('file', file.raw)
  uploadFile(formData).then((e) => {
    // 响参失败，提示错误消息
    if (e.success === false) {
      let message = e.message
      showMessage(message, 'error')
      return
    }

    // 成功则设置作者头像链接，并提示成功
    form.avatar = e.data.url
    showMessage('上传成功')
  })
}

// 保存当前博客设置
const onSubmit = () => {
  // 先验证 form 表单字段
  formRef.value.validate((valid) => {
    if (!valid) {
      console.log('表单验证不通过')
      return false
    }

    // 显示保存按钮 loading
    btnLoading.value = true
    updateBlogSettings(form).then((res) => {
      if (res.success === false) {
        // 获取服务端返回的错误消息
        let message = res.message
        // 提示错误消息
        showMessage(message, 'error')
        return
      }

      // 重新渲染页面中的信息
      initBlogSettings()
      showMessage('保存成功')
    }).finally(() => btnLoading.value = false) // 隐藏保存按钮 loading
  })
}

</script>


<style scoped>
/* 解决 textarea :focus 状态下，边框消失的问题 */
:deep(.el-textarea__inner:focus) {
  outline: 0 !important;
  box-shadow: 0 0 0 1px var(--el-color-primary) inset !important;
}

/* 覆盖 el-upload 默认样式，使其适应自定义容器 */
:deep(.el-upload) {
  width: 100%;
  height: 100%;
  display: block;
}

/* 去除默认边框，由外层 div 控制 */
:deep(.el-upload-dragger) {
  border: none;
  padding: 0;
  width: 100%;
  height: 100%;
}
</style>