<template>
  <div class="p-6">
    <!-- 顶部操作栏 (搜索 + 按钮) -->
    <SearchBar @search="getTableData" @reset="reset">
      <template #search-input>
        <el-input
            v-model="searchArticleTitle"
            clearable
            placeholder="请输入文章标题"
            prefix-icon="Search"
            @keyup.enter="getTableData"
        />
      </template>
      <template #date-picker>
        <el-date-picker
            v-model="pickDate"
            :shortcuts="shortcuts"
            end-placeholder="结束时间"
            range-separator="-"
            start-placeholder="开始时间"
            type="daterange"
            value-format="YYYY-MM-DD"
            @change="datepickerChange"
        />
      </template>
      <template #actions>
        <div class="flex items-center gap-2">
          <el-button plain @click="exportCSV" title="导出当前列表 CSV">
            <el-icon class="mr-1">
              <Download/>
            </el-icon>
            导出
          </el-button>
          <el-button :disabled="selectedRows.length === 0" type="danger" plain @click="batchDelete" title="批量删除选中项">
            <el-icon class="mr-1">
              <Delete/>
            </el-icon>
            批量删除
          </el-button>
          <el-button type="primary" @click="isArticlePublishEditorShow = true" title="新建文章">
            <el-icon class="mr-1">
              <EditPen/>
            </el-icon>
            写文章
          </el-button>
        </div>
      </template>
    </SearchBar>

    <div v-if="searchArticleTitle || (startDate.value && endDate.value)" class="mb-6">
      <div class="flex flex-wrap items-center gap-2">
        <span class="text-xs text-gray-400">已应用筛选：</span>
        <span v-if="searchArticleTitle"
          class="inline-flex items-center gap-1 px-2 py-1 rounded-full bg-gray-100 text-gray-600 text-xs">
          标题包含 “{{ searchArticleTitle }}”
        </span>
        <span v-if="startDate.value && endDate.value"
          class="inline-flex items-center gap-1 px-2 py-1 rounded-full bg-gray-100 text-gray-600 text-xs">
          时间 {{ startDate.value }} - {{ endDate.value }}
        </span>
        <el-button size="small" link type="primary" @click="reset">清除筛选</el-button>
      </div>
    </div>

    <!-- 表格区域 -->
    <el-card class="border-0 !bg-white !rounded-xl table-wrapper" shadow="never">
      <el-table v-loading="tableLoading" :data="tableData" :row-key="'id'" @selection-change="handleSelectionChange"
        :header-cell-style="{ background: '#f8fafc', color: '#64748b', fontWeight: '600' }" highlight-current-row
        style="width: 100%" empty-text="暂无文章，点击右上角写文章开始创作">
        <el-table-column type="selection" width="48" />
        <el-table-column label="封面" width="100">
          <template #default="scope">
            <div class="w-16 h-10 rounded overflow-hidden cursor-pointer group relative border border-gray-100">
              <el-image :preview-src-list="[scope.row.cover]" :src="scope.row.cover"
                class="w-full h-full object-cover transition-transform duration-300 group-hover:scale-110"
                hide-on-click-modal loading="lazy" preview-teleported>
                <template #error>
                  <div class="w-full h-full bg-gray-100 flex items-center justify-center text-gray-300">
                    <el-icon>
                      <Picture />
                    </el-icon>
                  </div>
                </template>
              </el-image>
            </div>
          </template>
        </el-table-column>

        <el-table-column label="标题" min-width="250" prop="title">
          <template #default="scope">
            <div class="font-medium text-gray-900 line-clamp-1 hover:text-sky-600 cursor-pointer transition-colors"
              @click="goArticleDetailPage(scope.row.id)">
              {{ scope.row.title }}
            </div>
          </template>
        </el-table-column>

        <el-table-column label="发布时间" prop="createTime" sortable width="180">
          <template #default="scope">
            <span class="text-gray-500 text-sm">{{ scope.row.createTime }}</span>
          </template>
        </el-table-column>

        <el-table-column label="状态" width="180">
          <template #default="scope">
            <div class="flex flex-col gap-2">
              <div class="flex items-center justify-between text-xs text-gray-500">
                <span>发布</span>
                <el-switch v-model="scope.row.isPublish" active-text="是" inactive-text="否" inline-prompt
                  style="--el-switch-on-color: #10b981;" @change="handleIsPublishChange(scope.row)" />
              </div>
              <div class="flex items-center justify-between text-xs text-gray-500">
                <span>仅登录</span>
                <el-switch v-model="scope.row.isPermission" active-text="开" inactive-text="关" inline-prompt
                  style="--el-switch-on-color: #f59e0b;" @change="handleIsPermissionChange(scope.row)" />
              </div>
            </div>
          </template>
        </el-table-column>

        <el-table-column fixed="right" label="操作" width="220">
          <template #default="scope">
            <div class="flex items-center gap-2">
              <el-button icon="Edit" link type="primary" @click="showArticleUpdateEditor(scope.row)">编辑</el-button>
              <el-button icon="View" link type="primary" @click="goArticleDetailPage(scope.row.id)">预览</el-button>
              <el-button icon="Delete" link type="danger" @click="deleteArticleSubmit(scope.row)">删除</el-button>
            </div>
          </template>
        </el-table-column>
      </el-table>

      <!-- 分页 -->
      <div class="mt-6 flex justify-end">
        <el-pagination v-model:current-page="current" v-model:page-size="size" :page-sizes="[10, 20, 50]" :total="total"
          background layout="total, sizes, prev, pager, next, jumper" @size-change="handleSizeChange"
          @current-change="getTableData" />
      </div>
    </el-card>

    <!-- 写文章弹窗 -->
    <el-dialog v-model="isArticlePublishEditorShow" :fullscreen="true" :show-close="false" class="no-header-dialog">
      <div class="h-full flex flex-col bg-gray-50">
        <!-- 顶部栏 -->
        <div
          class="h-14 bg-white border-b border-gray-200 flex items-center justify-between px-6 shrink-0 sticky top-0 z-50 shadow-sm">
          <div class="flex items-center gap-3">
            <el-button circle icon="ArrowLeft" @click="isArticlePublishEditorShow = false"></el-button>
            <h4 class="font-bold text-lg text-gray-800">写文章</h4>
          </div>
          <div class="flex items-center gap-3">
            <el-button @click="isArticlePublishEditorShow = false">取消</el-button>
            <el-button plain @click="publishPreview = true">预览</el-button>
            <el-button icon="Promotion" type="primary" @click="publishArticleSubmit">发布文章</el-button>
          </div>
        </div>

        <!-- 内容区 -->
        <div class="flex-1 overflow-y-auto p-6">
          <div class="px-0 sm:px-6 pb-3 text-xs text-gray-500 flex items-center justify-between">
            <span>自动保存：{{ autoSaveTimePublish || '未保存' }}</span>
            <span class="hidden sm:inline">快捷键：Ctrl+S 保存草稿 · Ctrl+Enter 发布 · Esc 关闭</span>
          </div>
          <div class="max-w-5xl mx-auto bg-white rounded-xl shadow-sm p-8 min-h-full">
            <el-form ref="publishArticleFormRef" :model="form" :rules="rules" label-position="top" size="large">
              <el-form-item label="文章标题" prop="title">
                <el-input v-model="form.title" maxlength="80" placeholder="请输入引人注目的标题..." show-word-limit />
              </el-form-item>

              <el-form-item label="文章内容" prop="content">
                <div class="border border-gray-200 rounded-lg overflow-hidden w-full">
                  <MdEditor v-model="form.content" editorId="publishArticleEditor" style="height: 70vh;"
                    @onUploadImg="onUploadImg" />
                </div>
              </el-form-item>

              <el-divider content-position="left">更多设置</el-divider>

              <div class="grid grid-cols-1 md:grid-cols-2 gap-8">
                <!-- 左侧 -->
                <div class="space-y-4">
                  <el-form-item label="文章封面" prop="cover">
                    <div class="flex flex-col gap-3 w-full">
                      <!-- 上传/预览区 -->
                      <div
                        class="w-full aspect-video bg-gray-50 border-2 border-dashed border-gray-200 rounded-lg flex items-center justify-center overflow-hidden hover:border-sky-400 transition-colors cursor-pointer relative group"
                        @click="$refs.uploadCoverInput.click()" @dragover.prevent @drop="handlePublishCoverDrop">
                        <img v-if="form.cover" :src="form.cover" class="w-full h-full object-cover" />
                        <div v-else class="flex flex-col items-center text-gray-400">
                          <el-icon size="32">
                            <Plus />
                          </el-icon>
                          <span class="text-xs mt-2">点击上传封面</span>
                        </div>
                        <!-- 悬停遮罩 -->
                        <div v-if="form.cover"
                          class="absolute inset-0 bg-black/40 hidden group-hover:flex items-center justify-center text-white font-medium">
                          更换封面
                        </div>
                      </div>

                      <!-- 隐藏的上传组件 -->
                      <div class="hidden">
                        <el-upload :auto-upload="false" :on-change="beforeUpload" :show-file-list="false"
                          accept="image/*" action="#">
                          <button ref="uploadCoverInput"></button>
                        </el-upload>
                      </div>

                      <!-- 历史图库选择 -->
                      <el-select v-model="form.cover" clearable placeholder="或从图库中选择" @change="handleSelectImage">
                        <el-option v-for="item in images" :key="item.value" :label="item.label" :value="item.value">
                          <div class="flex items-center gap-2">
                            <img :src="item.value" class="w-8 h-8 object-cover rounded border" />
                            <span class="truncate">{{ item.label }}</span>
                          </div>
                        </el-option>
                      </el-select>
                    </div>
                  </el-form-item>
                </div>

                <!-- 右侧 -->
                <div class="space-y-4">
                  <el-form-item label="文章分类" prop="categoryId">
                    <el-select v-model="form.categoryId" class="w-full" placeholder="请选择分类">
                      <el-option v-for="item in categories" :key="item.value" :label="item.label" :value="item.value" />
                    </el-select>
                  </el-form-item>

                  <el-form-item label="文章标签" prop="tags">
                    <el-select v-model="form.tags" :loading="tagSelectLoading" :remote-method="remoteMethod"
                      allow-create class="w-full" default-first-option filterable multiple placeholder="搜索或创建标签" remote>
                      <el-option v-for="item in tags" :key="item.value" :label="item.label" :value="item.value" />
                    </el-select>
                  </el-form-item>

                  <el-form-item label="文章摘要" prop="summary">
                    <el-input v-model="form.summary" :rows="4" maxlength="200" placeholder="请输入简短的文章摘要（可选）..."
                      show-word-limit type="textarea" />
                  </el-form-item>
                </div>
              </div>
            </el-form>
          </div>
          <div v-if="publishPreview" class="fixed inset-0 bg-black/60 z-[60] flex items-center justify-center p-4"
            @click.self="publishPreview = false">
            <div class="bg-white rounded-xl shadow-xl w-full max-w-5xl h-[80vh] overflow-hidden flex flex-col">
              <div class="flex items-center justify-between px-4 py-2 border-b">
                <h5 class="font-bold text-gray-800">预览</h5>
                <el-button size="small" @click="publishPreview = false">关闭</el-button>
              </div>
              <div class="flex-1 overflow-auto p-4">
                <MdPreview :modelValue="form.content" />
              </div>
            </div>
          </div>
        </div>
      </div>
    </el-dialog>


    <!-- 编辑博客弹窗 (复用写文章样式结构，仅数据源不同) -->
    <el-dialog v-model="isArticleUpdateEditorShow" :fullscreen="true" :show-close="false" class="no-header-dialog">
      <div class="h-full flex flex-col bg-gray-50">
        <!-- 顶部栏 -->
        <div
          class="h-14 bg-white border-b border-gray-200 flex items-center justify-between px-6 shrink-0 sticky top-0 z-50 shadow-sm">
          <div class="flex items-center gap-3">
            <el-button circle icon="ArrowLeft" @click="isArticleUpdateEditorShow = false"></el-button>
            <h4 class="font-bold text-lg text-gray-800">编辑文章</h4>
          </div>
          <div class="flex items-center gap-3">
            <el-button @click="isArticleUpdateEditorShow = false">取消</el-button>
            <el-button plain @click="updatePreview = true">预览</el-button>
            <el-button icon="Promotion" type="primary" @click="updateSubmit">保存修改</el-button>
          </div>
        </div>

        <!-- 内容区 -->
        <div class="flex-1 overflow-y-auto p-6">
          <div class="px-0 sm:px-6 pb-3 text-xs text-gray-500 flex items-center justify-between">
            <span>自动保存：{{ autoSaveTimeUpdate || '未保存' }}</span>
            <span class="hidden sm:inline">快捷键：Ctrl+S 保存草稿 · Ctrl+Enter 保存 · Esc 关闭</span>
          </div>
          <div class="max-w-5xl mx-auto bg-white rounded-xl shadow-sm p-8 min-h-full">
            <el-form ref="updateArticleFormRef" :model="updateArticleForm" :rules="rules" label-position="top"
              size="large">
              <el-form-item label="文章标题" prop="title">
                <el-input v-model="updateArticleForm.title" maxlength="80" placeholder="请输入引人注目的标题..."
                  show-word-limit />
              </el-form-item>

              <el-form-item label="文章内容" prop="content">
                <div class="border border-gray-200 rounded-lg overflow-hidden w-full">
                  <MdEditor v-model="updateArticleForm.content" editorId="updateArticleEditor" style="height: 70vh;"
                    @onUploadImg="onUploadImg" />
                </div>
              </el-form-item>

              <el-divider content-position="left">更多设置</el-divider>

              <div class="grid grid-cols-1 md:grid-cols-2 gap-8">
                <!-- 左侧 -->
                <div class="space-y-4">
                  <el-form-item label="文章封面" prop="cover">
                    <div class="flex flex-col gap-3 w-full">
                      <!-- 上传/预览区 -->
                      <div
                        class="w-full aspect-video bg-gray-50 border-2 border-dashed border-gray-200 rounded-lg flex items-center justify-center overflow-hidden hover:border-sky-400 transition-colors cursor-pointer relative group"
                        @click="$refs.updateUploadCoverInput.click()" @dragover.prevent @drop="handleUpdateCoverDrop">
                        <img v-if="updateArticleForm.cover" :src="updateArticleForm.coverUrl"
                          class="w-full h-full object-cover" alt="" />
                        <div v-else class="flex flex-col items-center text-gray-400">
                          <el-icon size="32">
                            <Plus />
                          </el-icon>
                          <span class="text-xs mt-2">点击上传封面</span>
                        </div>
                        <div v-if="updateArticleForm.cover"
                          class="absolute inset-0 bg-black/40 hidden group-hover:flex items-center justify-center text-white font-medium">
                          更换封面
                        </div>
                      </div>

                      <div class="hidden">
                        <el-upload :auto-upload="false" :on-change="beforeUpload" :show-file-list="false"
                          accept="image/*" action="#">
                          <button ref="updateUploadCoverInput"></button>
                        </el-upload>
                      </div>

                      <el-select v-model="updateArticleForm.cover" clearable placeholder="或从图库中选择"
                        @change="handleSelectImage">
                        <el-option v-for="item in images" :key="item.value" :label="item.label" :value="item.value">
                          <div class="flex items-center gap-2">
                            <img :src="item.value" class="w-8 h-8 object-cover rounded border" alt="" />
                            <span class="truncate">{{ item.label }}</span>
                          </div>
                        </el-option>
                      </el-select>
                    </div>
                  </el-form-item>
                </div>

                <!-- 右侧 -->
                <div class="space-y-4">
                  <el-form-item label="文章分类" prop="categoryId">
                    <el-select v-model="updateArticleForm.categoryId" class="w-full" placeholder="请选择分类">
                      <el-option v-for="item in categories" :key="item.value" :label="item.label" :value="item.value" />
                    </el-select>
                  </el-form-item>

                  <el-form-item label="文章标签" prop="tags">
                    <el-select v-model="updateArticleForm.tags" :loading="tagSelectLoading"
                      :remote-method="remoteMethod" allow-create class="w-full" default-first-option filterable multiple
                      placeholder="搜索或创建标签" remote>
                      <el-option v-for="item in tags" :key="item.value" :label="item.label" :value="item.value" />
                    </el-select>
                  </el-form-item>

                  <el-form-item label="文章摘要" prop="summary">
                    <el-input v-model="updateArticleForm.summary" :rows="4" maxlength="200"
                      placeholder="请输入简短的文章摘要（可选）..." show-word-limit type="textarea" />
                  </el-form-item>
                </div>
              </div>
            </el-form>
          </div>
          <div v-if="updatePreview" class="fixed inset-0 bg-black/60 z-[60] flex items-center justify-center p-4"
            @click.self="updatePreview = false">
            <div class="bg-white rounded-xl shadow-xl w-full max-w-5xl h-[80vh] overflow-hidden flex flex-col">
              <div class="flex items-center justify-between px-4 py-2 border-b">
                <h5 class="font-bold text-gray-800">预览</h5>
                <el-button size="small" @click="updatePreview = false">关闭</el-button>
              </div>
              <div class="flex-1 overflow-auto p-4">
                <MdPreview :modelValue="updateArticleForm.content" />
              </div>
            </div>
          </div>
        </div>
      </div>
    </el-dialog>

  </div>
</template>

<script setup>
import { reactive, ref, watch, onMounted, onBeforeUnmount } from 'vue'
import { EditPen, Picture, Plus, Download, Delete } from '@element-plus/icons-vue'
import moment from 'moment'
import {
  deleteArticle,
  getArticleDetail,
  getArticlePageList,
  publishArticle,
  updateArticle,
  updateArticleIsPermission,
  updateArticleIsPublish
} from "@/api/admin/article.js";
import { showMessage, showModel } from "@/composables/utils.js";
import { MdEditor, MdPreview } from 'md-editor-v3'
import 'md-editor-v3/lib/style.css'
import { getCategorySelectList } from "@/api/admin/category.js";
import { getTagSelectList, searchTags } from "@/api/admin/tag.js";
import { useRouter } from "vue-router";
import { findImageList, uploadFile } from "@/api/admin/file.js";
import SearchBar from "@/components/SearchBar.vue";

const router = useRouter()

// 跳转文章详情页
const goArticleDetailPage = (articleId) => {
  router.push('/article/' + articleId)
}

// 模糊搜索的文章标题
const searchArticleTitle = ref('')
// 日期
const pickDate = ref('')

// 是否显示文章发布对话框
const isArticlePublishEditorShow = ref(false)
const publishPreview = ref(false)
const autoSaveTimePublish = ref('')
const unsavedPublish = ref(false)

// 发布文章表单引用
const publishArticleFormRef = ref(null)

// 查询条件：开始结束时间
const startDate = reactive({})
const endDate = reactive({})

// 监听日期组件改变事件，并将开始结束时间设置到变量中
const datepickerChange = (e) => {
  if (e) {
    startDate.value = moment(e[0]).format('YYYY-MM-DD')
    endDate.value = moment(e[1]).format('YYYY-MM-DD')
  } else {
    startDate.value = null
    endDate.value = null
  }
}

const shortcuts = [
  {
    text: '最近一周',
    value: () => {
      const end = new Date()
      const start = new Date()
      start.setTime(start.getTime() - 3600 * 1000 * 24 * 7)
      return [start, end]
    },
  },
  {
    text: '最近一个月',
    value: () => {
      const end = new Date()
      const start = new Date()
      start.setTime(start.getTime() - 3600 * 1000 * 24 * 30)
      return [start, end]
    },
  },
  {
    text: '最近三个月',
    value: () => {
      const end = new Date()
      const start = new Date()
      start.setTime(start.getTime() - 3600 * 1000 * 24 * 90)
      return [start, end]
    },
  },
]

// 重置
const reset = () => {
  pickDate.value = ''
  startDate.value = null
  endDate.value = null
  searchArticleTitle.value = ''
  getTableData()
}

// 表格加载 Loading
const tableLoading = ref(false)
// 表格数据
const tableData = ref([])
// 当前页码，给了一个默认值 1
const current = ref(1)
// 总数据量，给了个默认值 0
const total = ref(0)
// 每页显示的数据量，给了个默认值 10
const size = ref(10)

// 每页展示数量变更事件
const handleSizeChange = (chooseSize) => {
  size.value = chooseSize
  getTableData()
}

// 表单对象
const form = reactive({
  id: null,
  title: '',
  content: '',
  cover: '',
  categoryId: null,
  tags: [],
  summary: ""
})

// 表单校验规则
const rules = {
  title: [
    { required: true, message: '请输入文章标题', trigger: 'blur' },
    { min: 1, max: 80, message: '文章标题要求大于1个字符，小于80个字符', trigger: 'blur' },
  ],
  content: [{ required: true, message: '文章内容不能为空', trigger: 'blur' }],
  cover: [{ required: true, message: '请上传文章封面', trigger: 'change' }],
  categoryId: [{ required: true, message: '请选择文章分类', trigger: 'change' }],
  tags: [{ required: true, message: '请选择文章标签', trigger: 'change' }],
}

// 更新发布状态
const handleIsPublishChange = (row) => {
  updateArticleIsPublish({ id: row.id, isPublish: row.isPublish }).then((res) => {
    // 重新请求分页接口，渲染列表数据
    // getTableData() // 可以不刷新，因为 v-model 已经变了

    if (res.success === false) {
      // 获取服务端返回的错误消息
      let message = res.message
      // 提示错误消息
      showMessage(message, 'error')
      // 回退状态
      row.isPublish = !row.isPublish
      return
    }

    showMessage(row.isPublish ? '发布成功' : "已取消发布")
  })
}

// 更新权限状态
const handleIsPermissionChange = (row) => {
  updateArticleIsPermission({ id: row.id, isPermission: row.isPermission }).then((res) => {
    // 重新请求分页接口，渲染列表数据
    // getTableData()

    if (res.success === false) {
      // 获取服务端返回的错误消息
      let message = res.message
      // 提示错误消息
      showMessage(message, 'error')
      // 回退状态
      row.isPermission = !row.isPermission
      return
    }

    showMessage(row.isPermission ? '权限校验已开启' : "权限校验已关闭")
  })
}

// 编辑器图片上传
const onUploadImg = async (files, callback) => {
  const res = await Promise.all(
    files.map((file) => {
      return new Promise((res, rej) => {
        console.log('==> 编辑器开始上传文件...')
        let formData = new FormData()
        formData.append("file", file);
        uploadFile(formData).then((res) => {
          console.log(res)
          console.log('访问路径：' + res.data.url)
          // 调用 callback 函数，回显上传图片
          callback([res.data.url]);
        })
      });
    })
  );
}

// 文章分类
const categories = ref([])
getCategorySelectList().then((e) => {
  categories.value = e.data
})

// 标签 select Loading 状态，默认不显示
const tagSelectLoading = ref(false)
// 文章标签
const tags = ref([])
// 渲染标签数据
getTagSelectList().then(res => {
  tags.value = res.data
})

// 根据用户输入的标签名称，远程模糊查询
const remoteMethod = (query) => {
  // 如果用户的查询关键词不为空
  if (query) {
    // 显示 loading
    tagSelectLoading.value = true
    // 调用标签模糊查询接口
    searchTags(query).then((e) => {
      if (e.success) {
        // 设置到 tags 变量中
        tags.value = e.data
      }
    }).finally(() => tagSelectLoading.value = false) // 隐藏 loading
  } else {
    // 为空时重置为初始列表（可选）
    getTagSelectList().then(res => {
      tags.value = res.data
    })
  }
}

// 编辑文章按钮点击事件
const showArticleUpdateEditor = (row) => {
  // 显示编辑文章对话框
  isArticleUpdateEditorShow.value = true
  findImage()
  // 拿到文章 ID
  let articleId = row.id
  getArticleDetail(articleId).then((res) => {
    if (res.success) {
      // 设置表单数据
      console.log(res.data)
      updateArticleForm.id = res.data.id
      updateArticleForm.title = res.data.title
      updateArticleForm.cover = res.data.cover
      updateArticleForm.coverUrl = res.data.coverUrl
      updateArticleForm.content = res.data.content
      updateArticleForm.categoryId = res.data.categoryId
      updateArticleForm.tags = res.data.tagIds
      updateArticleForm.summary = res.data.summary
    }
  })
}

// 发布文章
const publishArticleSubmit = () => {
  // 校验表单
  publishArticleFormRef.value.validate((valid) => {
    if (!valid) {
      return false
    }

    publishArticle(form).then((res) => {
      if (res.success === false) {
        // 获取服务端返回的错误消息
        let message = res.message
        // 提示错误消息
        showMessage(message, 'error')
        return
      }

      showMessage('发布成功')
      // 隐藏发布文章对话框
      isArticlePublishEditorShow.value = false
      // 将 form 表单字段置空
      form.title = ''
      form.content = ''
      form.cover = ''
      form.summary = ''
      form.categoryId = null
      form.tags = []
      // 重新请求分页接口，渲染列表数据
      getTableData()
    })
  })
}

// 保存文章
const updateSubmit = () => {
  updateArticleFormRef.value.validate((valid) => {
    // 校验表单
    if (!valid) {
      return false
    }

    // 请求更新文章接口
    updateArticle(updateArticleForm).then((res) => {
      if (res.success === false) {
        // 获取服务端返回的错误消息
        let message = res.message
        // 提示错误消息
        showMessage(message, 'error')
        return
      }

      showMessage('保存成功')
      // 隐藏编辑框
      isArticleUpdateEditorShow.value = false
      // 重新请求分页接口，渲染列表数据
      getTableData()
    })
  })
}

// 删除文章
const deleteArticleSubmit = (row) => {
  showModel('是否确定要删除该文章？').then(() => {
    deleteArticle(row.id).then((res) => {
      if (res.success === false) {
        // 获取服务端返回的错误消息
        let message = res.message
        // 提示错误消息
        showMessage(message, 'error')
        return
      }

      showMessage('删除成功')
      // 重新请求分页接口，渲染数据
      getTableData()
    })
  }).catch(() => {
    console.log('取消了')
  })
}

// 是否显示编辑文章对话框
const isArticleUpdateEditorShow = ref(false)
const updatePreview = ref(false)
const autoSaveTimeUpdate = ref('')
const unsavedUpdate = ref(false)
// 编辑文章表单引用
const updateArticleFormRef = ref(null)

// 修改文章表单对象
const updateArticleForm = reactive({
  id: null,
  title: '',
  content: '',
  cover: '',
  categoryId: null,
  tags: [],
  summary: ""
})

// 获取分页数据
function getTableData() {
  // 显示表格 loading
  tableLoading.value = true
  // 调用后台分页接口，并传入所需参数
  getArticlePageList({
    current: current.value,
    size: size.value,
    startDate: startDate.value,
    endDate: endDate.value,
    title: searchArticleTitle.value
  })
    .then((res) => {
      if (res.success === true) {
        tableData.value = res.data
        current.value = res.current
        size.value = res.size
        total.value = res.total
      }
    })
    .finally(() => tableLoading.value = false) // 隐藏表格 loading
}

getTableData()

// 选中项
const selectedRows = ref([])
const handleSelectionChange = (rows) => {
  selectedRows.value = rows || []
}

// 批量删除
const batchDelete = () => {
  if (!selectedRows.value.length) return
  showModel(`确定删除选中的 ${selectedRows.value.length} 篇文章吗？`).then(() => {
    const ids = selectedRows.value.map(r => r.id)
    Promise.all(ids.map(id => deleteArticle(id))).then(results => {
      const anyFail = results.some(r => r.success === false)
      if (anyFail) {
        showMessage('部分删除失败，请重试', 'error')
      } else {
        showMessage('批量删除成功')
      }
      getTableData()
    })
  }).catch(() => { })
}

// 导出 CSV
const exportCSV = () => {
  const rows = tableData.value || []
  const headers = ['ID', '标题', '发布时间', '已发布', '仅登录']
  const csvRows = [headers.join(',')]
  rows.forEach(r => {
    const line = [
      r.id,
      `"${(r.title || '').replace(/"/g, '""')}"`,
      r.createTime || '',
      r.isPublish ? '是' : '否',
      r.isPermission ? '开' : '关'
    ].join(',')
    csvRows.push(line)
  })
  const blob = new Blob([csvRows.join('\n')], { type: 'text/csv;charset=utf-8;' })
  const url = URL.createObjectURL(blob)
  const a = document.createElement('a')
  a.href = url
  a.download = `articles_${moment(new Date()).format('YYYYMMDD_HHmmss')}.csv`
  document.body.appendChild(a)
  a.click()
  document.body.removeChild(a)
  URL.revokeObjectURL(url)
}

const images = ref([])

function findImage() {
  findImageList()
    .then(res => {
      // 直接使用原始数据
      images.value = res.data.map(item => ({
        label: item.imageId,
        value: item.imageUrl
      }))
    })
}

findImage()

const uploadCoverUrl = ref('')

// 编辑文章：上传文章封面图片
const beforeUpload = (file) => {
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

    // 成功则设置表单对象中的封面链接，并提示上传成功
    // 判断当前是发布还是编辑
    if (isArticleUpdateEditorShow.value) {
      updateArticleForm.cover = e.data.url
    } else {
      form.cover = e.data.url
    }

    showMessage('上传成功')
  })
}

function handleSelectImage(val) {
  if (isArticleUpdateEditorShow.value) {
    updateArticleForm.cover = val
  } else {
    form.cover = val
  }
}

let autoSaveTimerPublish = null
let autoSaveTimerUpdate = null
function autoSavePublishDraft() {
  const data = {
    title: form.title,
    content: form.content,
    cover: form.cover,
    summary: form.summary,
    categoryId: form.categoryId,
    tags: form.tags,
    time: moment(new Date()).format('YYYY-MM-DD HH:mm:ss')
  }
  localStorage.setItem('article_publish_draft', JSON.stringify(data))
  autoSaveTimePublish.value = data.time
}
function loadPublishDraft() {
  const raw = localStorage.getItem('article_publish_draft')
  if (!raw) return
  try {
    const data = JSON.parse(raw)
    form.title = data.title || form.title
    form.content = data.content || form.content
    form.cover = data.cover || form.cover
    form.summary = data.summary || form.summary
    form.categoryId = data.categoryId || form.categoryId
    form.tags = data.tags || form.tags
    autoSaveTimePublish.value = data.time || ''
  } catch { }
}
function autoSaveUpdateDraft() {
  const key = `article_update_draft_${updateArticleForm.id || 'temp'}`
  const data = {
    id: updateArticleForm.id,
    title: updateArticleForm.title,
    content: updateArticleForm.content,
    cover: updateArticleForm.cover,
    summary: updateArticleForm.summary,
    categoryId: updateArticleForm.categoryId,
    tags: updateArticleForm.tags,
    time: moment(new Date()).format('YYYY-MM-DD HH:mm:ss')
  }
  localStorage.setItem(key, JSON.stringify(data))
  autoSaveTimeUpdate.value = data.time
}
function loadUpdateDraft() {
  const key = `article_update_draft_${updateArticleForm.id || 'temp'}`
  const raw = localStorage.getItem(key)
  if (!raw) return
  try {
    const data = JSON.parse(raw)
    updateArticleForm.title = data.title || updateArticleForm.title
    updateArticleForm.content = data.content || updateArticleForm.content
    updateArticleForm.cover = data.cover || updateArticleForm.cover
    updateArticleForm.summary = data.summary || updateArticleForm.summary
    updateArticleForm.categoryId = data.categoryId || updateArticleForm.categoryId
    updateArticleForm.tags = data.tags || updateArticleForm.tags
    autoSaveTimeUpdate.value = data.time || ''
  } catch { }
}

watch(form, () => {
  unsavedPublish.value = true
  clearTimeout(autoSaveTimerPublish)
  autoSaveTimerPublish = setTimeout(() => {
    autoSavePublishDraft()
    unsavedPublish.value = false
  }, 600)
}, { deep: true })

watch(updateArticleForm, () => {
  unsavedUpdate.value = true
  clearTimeout(autoSaveTimerUpdate)
  autoSaveTimerUpdate = setTimeout(() => {
    autoSaveUpdateDraft()
    unsavedUpdate.value = false
  }, 600)
}, { deep: true })

watch(isArticlePublishEditorShow, (open) => {
  if (open) loadPublishDraft()
})
watch(isArticleUpdateEditorShow, (open) => {
  if (open) loadUpdateDraft()
})

function handleKeyDown(e) {
  const ctrl = e.ctrlKey || e.metaKey
  if (isArticlePublishEditorShow.value) {
    if (ctrl && e.key === 's') {
      e.preventDefault()
      autoSavePublishDraft()
    } else if (ctrl && e.key === 'Enter') {
      e.preventDefault()
      publishArticleSubmit()
    } else if (e.key === 'Escape') {
      e.preventDefault()
      if (unsavedPublish.value) {
        showModel('存在未保存修改，确定关闭？').then(() => {
          isArticlePublishEditorShow.value = false
        }).catch(() => { })
      } else {
        isArticlePublishEditorShow.value = false
      }
    }
  } else if (isArticleUpdateEditorShow.value) {
    if (ctrl && e.key === 's') {
      e.preventDefault()
      autoSaveUpdateDraft()
    } else if (ctrl && e.key === 'Enter') {
      e.preventDefault()
      updateSubmit()
    } else if (e.key === 'Escape') {
      e.preventDefault()
      if (unsavedUpdate.value) {
        showModel('存在未保存修改，确定关闭？').then(() => {
          isArticleUpdateEditorShow.value = false
        }).catch(() => { })
      } else {
        isArticleUpdateEditorShow.value = false
      }
    }
  }
}
onMounted(() => {
  window.addEventListener('keydown', handleKeyDown)
})
onBeforeUnmount(() => {
  window.removeEventListener('keydown', handleKeyDown)
  clearTimeout(autoSaveTimerPublish)
  clearTimeout(autoSaveTimerUpdate)
})

function handlePublishCoverDrop(e) {
  const file = e.dataTransfer && e.dataTransfer.files && e.dataTransfer.files[0]
  if (!file) return
  const formData = new FormData()
  formData.append('file', file)
  uploadFile(formData).then((res) => {
    if (res.success === false) {
      let message = res.message
      showMessage(message, 'error')
      return
    }
    form.cover = res.data.url
    showMessage('上传成功')
  })
}
function handleUpdateCoverDrop(e) {
  const file = e.dataTransfer && e.dataTransfer.files && e.dataTransfer.files[0]
  if (!file) return
  const formData = new FormData()
  formData.append('file', file)
  uploadFile(formData).then((res) => {
    if (res.success === false) {
      let message = res.message
      showMessage(message, 'error')
      return
    }
    updateArticleForm.cover = res.data.url
    showMessage('上传成功')
  })
}
</script>

<style scoped>
/* 覆盖表格悬停样式，使其更柔和 */
:deep(.el-table--enable-row-hover .el-table__body tr:hover > td.el-table__cell) {
  background-color: #f8fafc;
}

/* 弹窗无 Header 模式样式调整 */
:deep(.no-header-dialog .el-dialog__header) {
  display: none;
  padding: 0;
  margin: 0;
}

:deep(.no-header-dialog .el-dialog__body) {
  padding: 0;
  height: 100%;
}

/* 隐藏 input file */
input[type="file"] {
  display: none;
}
</style>
