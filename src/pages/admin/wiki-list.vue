<template>
  <div class="p-6">
    <!-- 顶部操作栏 -->
    <SearchBar @search="getTableData" @reset="reset">
      <template #search-input>
        <el-input
            v-model="searchWikiTitle"
            clearable
            placeholder="请输入知识库标题"
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
        <el-button icon="Plus" type="primary" @click="addWikiBtnClick">新增知识库</el-button>
      </template>
    </SearchBar>

    <!-- 表格区域 -->
    <el-card class="border-0 !bg-white !rounded-xl table-wrapper" shadow="never">
      <el-table
          v-loading="tableLoading"
          :data="tableData"
          :header-cell-style="{ background: '#f8fafc', color: '#64748b', fontWeight: '600' }"
          highlight-current-row
          style="width: 100%"
      >
        <el-table-column label="序号" type="index" width="60"/>

        <el-table-column label="封面" width="100">
          <template #default="scope">
            <div class="w-16 h-10 rounded overflow-hidden cursor-pointer group relative border border-gray-100">
              <el-image
                  :preview-src-list="[scope.row.cover]"
                  :src="scope.row.cover"
                  class="w-full h-full object-cover transition-transform duration-300 group-hover:scale-110"
                  hide-on-click-modal
                  loading="lazy"
                  preview-teleported
              >
                <template #error>
                  <div class="w-full h-full bg-gray-100 flex items-center justify-center text-gray-300">
                    <el-icon>
                      <Picture/>
                    </el-icon>
                  </div>
                </template>
              </el-image>
            </div>
          </template>
        </el-table-column>

        <el-table-column label="标题" min-width="200" prop="title">
          <template #default="scope">
            <span class="font-medium text-gray-700">{{ scope.row.title }}</span>
          </template>
        </el-table-column>

        <el-table-column label="发布时间" prop="createTime" sortable width="180">
          <template #default="scope">
            <div class="flex items-center gap-1 text-gray-500 text-sm">
              <el-icon>
                <Clock/>
              </el-icon>
              <span>{{ scope.row.createTime }}</span>
            </div>
          </template>
        </el-table-column>

        <el-table-column label="状态设置" width="180">
          <template #default="scope">
            <div class="flex flex-col gap-2">
              <div class="flex items-center justify-between text-xs text-gray-500">
                <span>置顶</span>
                <el-switch
                    v-model="scope.row.isTop"
                    active-text="是"
                    inactive-text="否"
                    inline-prompt
                    style="--el-switch-on-color: #f59e0b;"
                    @change="handleIsTopChange(scope.row)"
                />
              </div>
              <div class="flex items-center justify-between text-xs text-gray-500">
                <span>发布</span>
                <el-switch
                    v-model="scope.row.isPublish"
                    active-text="是"
                    inactive-text="否"
                    inline-prompt
                    style="--el-switch-on-color: #10b981;"
                    @change="handleIsPublishChange(scope.row)"
                />
              </div>
            </div>
          </template>
        </el-table-column>

        <el-table-column fixed="right" label="操作" width="200">
          <template #default="scope">
            <div class="flex items-center gap-2">
              <el-tooltip :show-after="500" content="编辑基本信息" placement="top">
                <el-button icon="Edit" link type="primary" @click="showEditWikiDialog(scope.row)"></el-button>
              </el-tooltip>

              <el-tooltip :show-after="500" content="编辑目录结构" placement="top">
                <el-button icon="Tickets" link type="warning" @click="showEditWikiCatalogDialog(scope.row)"></el-button>
              </el-tooltip>

              <el-tooltip :show-after="500" content="预览" placement="top">
                <el-button icon="View" link type="info"></el-button>
              </el-tooltip>

              <el-popconfirm title="确定删除该知识库吗？此操作不可恢复。" @confirm="deleteWikiSubmit(scope.row)">
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

    <!-- 新增知识库弹窗 -->
    <FormDialog ref="formDialogRef" destroyOnClose title="新增知识库" width="600px" @submit="onSubmit">
      <el-form ref="formRef" :model="form" :rules="rules" label-position="top">
        <div class="flex gap-6">
          <!-- 左侧封面上传 -->
          <div class="w-1/3">
            <el-form-item label="封面" prop="cover">
              <el-upload
                  :auto-upload="false"
                  :on-change="handleCoverChange"
                  :show-file-list="false"
                  accept="image/*"
                  action="#"
                  class="w-full"
              >
                <div
                    class="w-full aspect-[3/4] rounded-lg border-2 border-dashed border-gray-300 hover:border-sky-500 flex flex-col items-center justify-center cursor-pointer bg-gray-50 transition-colors overflow-hidden group">
                  <img v-if="form.cover" :src="form.cover" class="w-full h-full object-cover"/>
                  <div v-else class="flex flex-col items-center text-gray-400">
                    <el-icon size="24">
                      <Plus/>
                    </el-icon>
                    <span class="text-xs mt-2">上传封面</span>
                  </div>
                  <div v-if="form.cover"
                       class="absolute inset-0 bg-black/40 hidden group-hover:flex items-center justify-center text-white text-xs">
                    更换图片
                  </div>
                </div>
              </el-upload>
            </el-form-item>
          </div>

          <!-- 右侧信息 -->
          <div class="flex-1">
            <el-form-item label="标题" prop="title">
              <el-input v-model="form.title" clearable maxlength="20" placeholder="请输入知识库标题" show-word-limit/>
            </el-form-item>
            <el-form-item label="摘要" prop="summary">
              <el-input v-model="form.summary" :rows="4" clearable maxlength="50" placeholder="请输入知识库摘要..."
                        show-word-limit type="textarea"/>
            </el-form-item>
          </div>
        </div>
      </el-form>
    </FormDialog>

    <!-- 编辑知识库弹窗 -->
    <FormDialog ref="editFormDialogRef" destroyOnClose title="编辑知识库" width="600px" @submit="onEditWikiSubmit">
      <el-form ref="editFormRef" :model="editForm" :rules="rules" label-position="top">
        <div class="flex gap-6">
          <!-- 左侧封面上传 -->
          <div class="w-1/3">
            <el-form-item label="封面" prop="cover">
              <el-upload
                  :auto-upload="false"
                  :on-change="handleUpdateCoverChange"
                  :show-file-list="false"
                  accept="image/*"
                  action="#"
                  class="w-full"
              >
                <div
                    class="w-full aspect-[3/4] rounded-lg border-2 border-dashed border-gray-300 hover:border-sky-500 flex flex-col items-center justify-center cursor-pointer bg-gray-50 transition-colors overflow-hidden group">
                  <img v-if="editForm.cover" :src="editForm.cover" class="w-full h-full object-cover"/>
                  <div v-else class="flex flex-col items-center text-gray-400">
                    <el-icon size="24">
                      <Plus/>
                    </el-icon>
                    <span class="text-xs mt-2">上传封面</span>
                  </div>
                  <div v-if="editForm.cover"
                       class="absolute inset-0 bg-black/40 hidden group-hover:flex items-center justify-center text-white text-xs">
                    更换图片
                  </div>
                </div>
              </el-upload>
            </el-form-item>
          </div>

          <!-- 右侧信息 -->
          <div class="flex-1">
            <el-form-item label="标题" prop="title">
              <el-input v-model="editForm.title" clearable maxlength="20" placeholder="请输入知识库标题"
                        show-word-limit/>
            </el-form-item>
            <el-form-item label="摘要" prop="summary">
              <el-input v-model="editForm.summary" :rows="4" clearable maxlength="50" placeholder="请输入知识库摘要..."
                        show-word-limit type="textarea"/>
            </el-form-item>
          </div>
        </div>
      </el-form>
    </FormDialog>

    <!-- 目录编辑 -->
    <WikiCatalogEditDialog ref="editCatalogFormDialogRef" destroyOnClose title="编辑目录" width="70%">
    </WikiCatalogEditDialog>

  </div>
</template>

<script setup>
import {reactive, ref} from 'vue'
import {Clock, Picture, Plus} from '@element-plus/icons-vue'
import {
  addWiki,
  deleteWiki,
  getWikiPageList,
  updateWiki,
  updateWikiIsPublish,
  updateWikiIsTop
} from "@/api/admin/wiki.js"
import moment from 'moment'
import FormDialog from "@/components/FormDialog.vue";
import {showMessage} from "@/composables/utils.js";
import {uploadFile} from "@/api/admin/file.js";
import WikiCatalogEditDialog from "@/components/WikiCatalogEditDialog.vue";
import SearchBar from "@/components/SearchBar.vue";

// 编辑目录对话框是否显示
const editCatalogFormDialogRef = ref(null)

// 模糊搜索的知识库标题
const searchWikiTitle = ref('')
// 日期
const pickDate = ref('')
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
  searchWikiTitle.value = ''
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


// 获取分页数据
function getTableData() {
  // 显示表格 loading
  tableLoading.value = true
  // 调用后台分页接口，并传入所需参数
  getWikiPageList({
    current: current.value,
    size: size.value,
    startDate: startDate.value,
    endDate: endDate.value,
    title: searchWikiTitle.value
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

// 每页展示数量变更事件
const handleSizeChange = (chooseSize) => {
  size.value = chooseSize
  getTableData()
}

// 对话框引用
const formDialogRef = ref(null)

// 新增知识库按钮点击事件, 弹出 Dialog
const addWikiBtnClick = () => {
  formDialogRef.value.open()
}

// 表单引用
const formRef = ref(null)
// 表单对象
const form = reactive({
  title: '',
  cover: '',
  summary: ''
})

// 表单校验规则
const rules = {
  title: [
    {required: true, message: '请输入标题', trigger: 'blur'},
    {min: 1, max: 20, message: '标题要求大于1个字符，小于20个字符', trigger: 'blur'},
  ],
  summary: [
    {required: true, message: '请输入摘要', trigger: 'blur'},
    {min: 1, max: 50, message: '摘要要求大于1个字符，小于50个字符', trigger: 'blur'},
  ],
  cover: [{required: true, message: '请上传封面', trigger: 'change'}],
}

// 上传封面图片
const handleCoverChange = (file) => {
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
    form.cover = e.data.url
    showMessage('上传成功')
  })
}

const onSubmit = () => {
  // 先验证 form 表单字段
  formRef.value.validate((valid) => {
    if (!valid) {
      return false
    }

    // 显示提交按钮 loading
    formDialogRef.value.showBtnLoading()
    addWiki(form).then((res) => {
      if (!res.success) {
        // 获取服务端返回的错误消息
        let message = res.message
        // 提示错误消息
        showMessage(message, 'error')
        return
      }

      showMessage('添加成功')
      // 将表单中数据置空
      form.title = ''
      form.cover = ''
      form.summary = ''
      // 隐藏对话框
      formDialogRef.value.close()
      // 重新请求分页接口，渲染数据
      getTableData()
    }).finally(() => formDialogRef.value.closeBtnLoading()) // 隐藏提交按钮 loading

  })
}

// 更新发布状态
const handleIsPublishChange = (row) => {
  updateWikiIsPublish({id: row.id, isPublish: row.isPublish}).then((res) => {
    // 重新请求分页接口，渲染列表数据
    // getTableData()

    if (res.success === false) {
      // 获取服务端返回的错误消息
      let message = res.message
      // 提示错误消息
      showMessage(message, 'error')
      // 回滚
      row.isPublish = !row.isPublish
      return
    }

    showMessage(row.isPublish ? '发布成功' : "已取消发布")
  })
}

// 更新置顶
const handleIsTopChange = (row) => {
  updateWikiIsTop({id: row.id, isTop: row.isTop}).then((res) => {
    // 重新请求分页接口，渲染列表数据
    // getTableData()

    if (res.success === false) {
      // 获取服务端返回的错误消息
      let message = res.message
      // 提示错误消息
      showMessage(message, 'error')
      // 回滚
      row.isTop = !row.isTop
      return
    }

    showMessage(row.isTop ? '置顶成功' : "已取消置顶")
  })
}

// 删除知识库
const deleteWikiSubmit = (row) => {
  deleteWiki(row.id).then((res) => {
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
}

// 更新知识库对话框引用
const editFormDialogRef = ref(null)
// 弹出知识库编辑对话框

// 弹出知识库编辑对话框
const showEditWikiDialog = (row) => {
  editFormDialogRef.value.open()
  editForm.id = row.id
  editForm.title = row.title
  editForm.cover = row.cover
  editForm.summary = row.summary
}

// 表单引用
const editFormRef = ref(null)
// 表单对象
const editForm = reactive({
  id: null,
  title: '',
  cover: '',
  summary: ''
})

// 知识库编辑：上传封面图片
const handleUpdateCoverChange = (file) => {
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
    editForm.cover = e.data.url
    showMessage('上传成功')
  })
}

// 编辑知识库提交事件
const onEditWikiSubmit = () => {
  // 先验证 form 表单字段
  editFormRef.value.validate((valid) => {
    if (!valid) {
      return false
    }

    // 显示提交按钮 loading
    editFormDialogRef.value.showBtnLoading()
    updateWiki(editForm).then((res) => {
      if (!res.success) {
        // 获取服务端返回的错误消息
        let message = res.message
        // 提示错误消息
        showMessage(message, 'error')
        return
      }

      showMessage('更新成功')
      // 将编辑表单中数据置空
      editForm.id = null
      editForm.title = ''
      editForm.cover = ''
      editForm.summary = ''
      // 隐藏对话框
      editFormDialogRef.value.close()
      // 重新请求分页接口，渲染数据
      getTableData()
    }).finally(() => editFormDialogRef.value.closeBtnLoading()) // 隐藏提交按钮 loading

  })
}

// 编辑目录按钮点击事件
const showEditWikiCatalogDialog = (row) => {
  // 显示编辑目录对话框, 并传入知识库 ID
  editCatalogFormDialogRef.value.open(row.id)
}
</script>

<style scoped>
/* 覆盖表格悬停样式 */
:deep(.el-table--enable-row-hover .el-table__body tr:hover > td.el-table__cell) {
  background-color: #f8fafc;
}
</style>
