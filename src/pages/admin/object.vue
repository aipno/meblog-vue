<template>
  <div class="p-6">
    <!-- 顶部操作栏 -->
    <div class="flex flex-wrap items-center justify-between mb-6 gap-4">
      <!-- 左侧搜索区 -->
      <div class="flex items-center gap-3 flex-wrap">
        <div class="w-64">
          <el-input
              v-model="searchFileName"
              clearable
              placeholder="请输入文件名"
              prefix-icon="Search"
              @keyup.enter="getTableData"
          />
        </div>

        <div class="w-80">
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
        </div>

        <el-button icon="Search" plain type="primary" @click="getTableData">搜索</el-button>
        <el-button icon="RefreshRight" @click="reset">重置</el-button>
      </div>

      <!-- 右侧操作区 -->
      <div>
        <el-button icon="Upload" type="primary" @click="uploadFileBtnClick">上传文件</el-button>
      </div>
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
        <el-table-column label="预览" width="120">
          <template #default="scope">
            <div
                class="flex items-center justify-center h-16 w-24 bg-gray-50 rounded-lg overflow-hidden border border-gray-100 cursor-pointer group">
              <el-image
                  v-if="isImage(scope.row.fileName)"
                  :initial-index="getImageIndex(scope.row)"
                  :preview-src-list="getImagePreviewList()"
                  :src="getImageSrc(scope.row)"
                  class="w-full h-full object-cover transition-transform duration-300 group-hover:scale-110"
                  fit="cover"
                  hide-on-click-modal
                  loading="lazy"
                  preview-teleported
              >
                <template #error>
                  <div class="flex flex-col items-center justify-center text-gray-400">
                    <el-icon size="20">
                      <Picture/>
                    </el-icon>
                  </div>
                </template>
              </el-image>
              <div v-else class="flex flex-col items-center justify-center text-gray-400">
                <!-- 根据文件类型显示不同图标 -->
                <el-icon class="group-hover:scale-110 transition-transform" size="28">
                  <Document/>
                </el-icon>
              </div>
            </div>
          </template>
        </el-table-column>

        <el-table-column label="文件名" min-width="200" prop="fileName">
          <template #default="scope">
            <div :title="scope.row.fileName" class="font-medium text-gray-700 truncate">{{ scope.row.fileName }}</div>
            <div class="text-xs text-gray-400 mt-1">{{ scope.row.contentType }}</div>
          </template>
        </el-table-column>

        <el-table-column label="大小" prop="fileSize" sortable width="120">
          <template #default="scope">
            <span class="text-sm text-gray-600 font-mono bg-gray-100 px-2 py-0.5 rounded">{{
                formatFileSize(scope.row.fileSize)
              }}</span>
          </template>
        </el-table-column>

        <el-table-column fixed="right" label="操作" width="180">
          <template #default="scope">
            <div class="flex items-center gap-2">
              <el-tooltip :show-after="500" content="下载" placement="top">
                <el-button icon="Download" link type="primary" @click="downloadFile(scope.row)"></el-button>
              </el-tooltip>
              <el-tooltip :show-after="500" content="复制链接" placement="top">
                <el-button icon="CopyDocument" link type="primary" @click="copyLink(scope.row)"></el-button>
              </el-tooltip>
              <el-popconfirm title="确定永久删除该文件吗？" @confirm="deleteFileSubmit(scope.row)">
                <template #reference>
                  <el-button icon="Delete" link type="danger"></el-button>
                </template>
              </el-popconfirm>
            </div>
          </template>
        </el-table-column>
      </el-table>

      <!-- 分页 -->
      <div class="mt-6 flex justify-end">
        <el-pagination
            v-model:current-page="current"
            v-model:page-size="size"
            :page-sizes="[10, 20, 50]"
            :total="total"
            background
            layout="total, sizes, prev, pager, next, jumper"
            @size-change="handleSizeChange"
            @current-change="getTableData"
        />
      </div>
    </el-card>

    <!-- 上传文件弹窗 -->
    <FormDialog ref="uploadDialogRef" :showCancel="false" :showConfirm="false" destroyOnClose title="上传文件"
                width="500px">
      <div class="p-4">
        <el-upload
            ref="uploadRef"
            :auto-upload="false"
            :limit="1"
            :on-change="handleFileChange"
            :on-exceed="handleExceed"
            :on-remove="handleRemove"
            accept="*/*"
            action="#"
            class="upload-demo w-full"
            drag
        >
          <div class="py-8 flex flex-col items-center justify-center">
            <el-icon class="el-icon--upload text-6xl text-sky-200 mb-4">
              <upload-filled/>
            </el-icon>
            <div class="el-upload__text text-gray-600">
              拖拽文件到此处，或 <em class="text-sky-600 font-medium">点击上传</em>
            </div>
            <div class="text-xs text-gray-400 mt-2">支持任意格式文件，单个文件不超过 100MB</div>
          </div>
        </el-upload>

        <!-- 底部按钮区 -->
        <div class="flex justify-end gap-3 mt-6 border-t border-gray-100 pt-4">
          <el-button @click="closeUploadDialog">取消</el-button>
          <el-button :disabled="!fileList.length" icon="Upload" type="primary" @click="submitUpload">开始上传
          </el-button>
        </div>
      </div>
    </FormDialog>

  </div>
</template>

<script setup>
import {Document, Picture, UploadFilled} from '@element-plus/icons-vue'
import {reactive, ref} from 'vue'
import moment from 'moment'
import {showMessage} from '@/composables/utils'
import {objectList, uploadFile} from '@/api/admin/file'
import FormDialog from '@/components/FormDialog.vue'

// 搜索相关
const searchFileName = ref('')
const pickDate = ref('')
const startDate = reactive({})
const endDate = reactive({})

// 表格相关
const tableLoading = ref(false)
const tableData = ref([])
const current = ref(1)
const total = ref(0)
const size = ref(10)

// 上传相关
const uploadDialogRef = ref(null)
const uploadRef = ref()
const fileList = ref([])

// 日期选择器
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
    text: '最近一周', value: () => {
      const end = new Date();
      const start = new Date();
      start.setTime(start.getTime() - 3600 * 1000 * 24 * 7);
      return [start, end]
    }
  },
  {
    text: '最近一个月', value: () => {
      const end = new Date();
      const start = new Date();
      start.setTime(start.getTime() - 3600 * 1000 * 24 * 30);
      return [start, end]
    }
  },
  {
    text: '最近三个月', value: () => {
      const end = new Date();
      const start = new Date();
      start.setTime(start.getTime() - 3600 * 1000 * 24 * 90);
      return [start, end]
    }
  },
]

// 分页变更
const handleSizeChange = (chooseSize) => {
  size.value = chooseSize
  getTableData()
}

// 重置
const reset = () => {
  searchFileName.value = ''
  pickDate.value = ''
  startDate.value = null
  endDate.value = null
  getTableData()
}

// 获取数据
function getTableData() {
  tableLoading.value = true
  objectList()
      .then(res => {
        if (res && res.data && Array.isArray(res.data)) {
          // 这里如果是真实后端分页，应该传参。目前假设是模拟数据或全量返回
          // 如果是全量返回，前端做假分页（可选），或者后端接口其实已经支持分页参数
          // 假设后端接口目前不支持分页参数，这里简单展示所有数据
          tableData.value = res.data.map(item => ({
            id: item.id,
            fileName: item.fileName,
            fileSize: item.fileSize,
            contentType: item.contentType,
            previewData: item.previewData,
            url: item.url,
          }))
          total.value = res.data.length
        } else {
          tableData.value = []
          total.value = 0
        }
      }).catch(error => {
    showMessage(error.message || '操作失败', 'error')
    tableData.value = []
  }).finally(() => {
    tableLoading.value = false
  })
}

getTableData()

// 上传弹窗
const uploadFileBtnClick = () => {
  uploadDialogRef.value.open()
  fileList.value = []
}

const closeUploadDialog = () => {
  uploadDialogRef.value.close()
}

// 文件选择
const handleFileChange = (file, fileListArg) => {
  fileList.value = fileListArg
}

const handleRemove = (file, fileListArg) => {
  fileList.value = fileListArg
}

const handleExceed = () => {
  showMessage('单次只能上传一个文件，请移除后重试', 'warning')
}

// 提交上传
const submitUpload = () => {
  if (!uploadRef.value) return
  if (fileList.value.length === 0) {
    showMessage('请选择要上传的文件', 'warning')
    return
  }

  // 模拟上传过程，显示 loading
  uploadDialogRef.value.showBtnLoading() // 如果 FormDialog 支持这个方法

  uploadFile(fileList.value[0].raw)
      .then(res => {
        if (res.success) {
          showMessage('上传成功')
          closeUploadDialog()
          getTableData()
          fileList.value = [] // 清空列表
          uploadRef.value.clearFiles()
        } else {
          showMessage(res.message || '上传失败', 'error')
        }
      })
      .catch(err => {
        showMessage(err.message || '上传失败', 'error')
      })
      .finally(() => {
        if (uploadDialogRef.value.closeBtnLoading) uploadDialogRef.value.closeBtnLoading()
      })
}

// 下载
const downloadFile = (row) => {
  if (row.url) {
    window.open(row.url, '_blank')
  } else {
    showMessage('文件链接无效', 'warning')
  }
}

// 复制链接
const copyLink = (row) => {
  if (!row.url) {
    showMessage('链接为空', 'warning')
    return
  }
  navigator.clipboard.writeText(row.url).then(() => {
    showMessage('链接已复制到剪贴板')
  }).catch(() => {
    showMessage('复制失败，请手动复制', 'error')
  })
}

// 删除
const deleteFileSubmit = (row) => {
  // 模拟删除
  showMessage('演示环境，删除功能暂未开放', 'warning')
  // 真实逻辑：
  // deleteFile(row.id).then(...)
}

// 工具函数
const formatFileSize = (size) => {
  if (size === null || size === undefined || isNaN(size)) return '0 B'
  if (size < 1024) return size + ' B'
  else if (size < 1024 * 1024) return (size / 1024).toFixed(2) + ' KB'
  else if (size < 1024 * 1024 * 1024) return (size / (1024 * 1024)).toFixed(2) + ' MB'
  else return (size / (1024 * 1024 * 1024)).toFixed(2) + ' GB'
}

const isImage = (fileName) => {
  if (!fileName || typeof fileName !== 'string') return false
  const imageExtensions = ['jpg', 'jpeg', 'png', 'gif', 'bmp', 'webp', 'svg', 'ico']
  const extension = fileName.split('.').pop().toLowerCase()
  return imageExtensions.includes(extension)
}

const getImageSrc = (row) => {
  if (!row) return ''
  if (row.url) return row.url
  if (row.previewData && row.contentType) return `data:${row.contentType};base64,${row.previewData}`
  return ''
}

const getImagePreviewList = () => {
  if (!tableData.value) return []
  return tableData.value.filter(item => isImage(item.fileName)).map(item => getImageSrc(item))
}

const getImageIndex = (row) => {
  const images = tableData.value.filter(item => isImage(item.fileName))
  return images.findIndex(item => item.id === row.id)
}
</script>

<style scoped>
/* 覆盖表格悬停样式 */
:deep(.el-table--enable-row-hover .el-table__body tr:hover > td.el-table__cell) {
  background-color: #f8fafc;
}

/* 上传组件样式微调 */
:deep(.el-upload-dragger) {
  border-radius: 0.75rem;
  border-color: #e5e7eb;
  transition: all 0.3s;
}

:deep(.el-upload-dragger:hover) {
  border-color: var(--el-color-primary);
  background-color: #f0f9ff;
}
</style>