<template>
  <div class="p-6">
    <!-- 顶部操作栏 -->
    <div class="flex flex-wrap items-center justify-between mb-6 gap-4">
      <!-- 左侧搜索区 -->
      <div class="flex items-center gap-3 flex-wrap">
        <div class="w-64">
          <el-input
              v-model="searchTagName"
              clearable
              placeholder="请输入标签名称"
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
        <el-button icon="Plus" type="primary" @click="addCategoryBtnClick">新增标签</el-button>
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
        <el-table-column label="标签名称" min-width="180" prop="name">
          <template #default="scope">
            <el-tag class="ml-2" effect="light" round type="primary">
              <div class="flex items-center gap-1">
                <el-icon>
                  <PriceTag/>
                </el-icon>
                <span>{{ scope.row.name }}</span>
              </div>
            </el-tag>
          </template>
        </el-table-column>

        <el-table-column label="文章数" prop="articlesTotal" sortable width="120">
          <template #default="scope">
            <el-tag effect="plain" round type="info">{{ scope.row.articlesTotal || 0 }} 篇</el-tag>
          </template>
        </el-table-column>

        <el-table-column label="创建时间" prop="createTime" sortable width="180">
          <template #default="scope">
            <div class="flex items-center gap-1 text-gray-500 text-sm">
              <el-icon>
                <Clock/>
              </el-icon>
              <span>{{ scope.row.createTime }}</span>
            </div>
          </template>
        </el-table-column>

        <el-table-column fixed="right" label="操作" width="120">
          <template #default="scope">
            <el-popconfirm title="确定删除该标签吗？" @confirm="deleteTagSubmit(scope.row)">
              <template #reference>
                <el-button icon="Delete" link type="danger">删除</el-button>
              </template>
            </el-popconfirm>
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

    <!-- 添加标签弹窗 -->
    <FormDialog ref="formDialogRef" destroyOnClose title="添加文章标签" width="480px" @submit="onSubmit">
      <el-form ref="formRef" :model="form" label-position="top">
        <el-form-item label="标签名称" prop="name">
          <div
              class="flex items-center gap-2 flex-wrap p-3 border border-gray-200 rounded-lg min-h-[60px] bg-gray-50/50 cursor-text"
              @click="showInput">
            <el-tag
                v-for="tag in dynamicTags"
                :key="tag"
                :disable-transitions="false"
                class="mx-1"
                closable
                effect="light"
                type="primary"
                @close="handleClose(tag)"
            >
              {{ tag }}
            </el-tag>
            <el-input
                v-if="inputVisible"
                ref="InputRef"
                v-model="inputValue"
                class="w-24 ml-1 !h-6"
                placeholder="输入"
                size="small"
                @blur="handleInputConfirm"
                @keyup.enter="handleInputConfirm"
            />
            <el-button v-else class="button-new-tag ml-1 !h-6" icon="Plus" link size="small" type="primary"
                       @click.stop="showInput">
              新标签
            </el-button>
          </div>
          <div class="text-xs text-gray-400 mt-2 flex items-center gap-1">
            <el-icon>
              <InfoFilled/>
            </el-icon>
            <span>支持批量添加，输入标签名后按回车确认。</span>
          </div>
        </el-form-item>
      </el-form>
    </FormDialog>

  </div>
</template>

<script setup>
import {Clock, InfoFilled, PriceTag} from '@element-plus/icons-vue'
import {nextTick, reactive, ref} from 'vue'
import {addTag, deleteTag, getTagPageList} from '@/api/admin/tag'
import moment from 'moment'
import {showMessage} from '@/composables/utils'
import FormDialog from '@/components/FormDialog.vue'

// 分页查询的标签名称s
const searchTagName = ref('')
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

  getTagPageList({
    current: current.value,
    size: size.value,
    startDate: startDate.value,
    endDate: endDate.value,
    name: searchTagName.value
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
  console.log('选择的页码' + chooseSize)
  size.value = chooseSize
  getTableData()
}

// 重置查询条件
const reset = () => {
  searchTagName.value = ''
  pickDate.value = ''
  startDate.value = null
  endDate.value = null
  getTableData()
}

// 对话框是否显示
const formDialogRef = ref(null)

// 新增分类按钮点击事件
const addCategoryBtnClick = () => {
  formDialogRef.value.open()
}


// 表单引用
const formRef = ref(null)

// 添加文章分类表单对象
const form = reactive({
  tags: []
})


const onSubmit = () => {
  // 校验是否添加了标签
  if (dynamicTags.value.length === 0) {
    showMessage('请至少添加一个标签', 'warning')
    return
  }

  // 显示提交按钮 loading
  formDialogRef.value.showBtnLoading()
  form.tags = dynamicTags.value

  addTag(form).then((res) => {
    if (res.success === true) {
      showMessage('添加成功')
      // 将表单中标签数组置空
      form.tags = []
      dynamicTags.value = []
      // 隐藏对话框
      formDialogRef.value.close()
      // 重新请求分页接口，渲染数据
      getTableData()
    } else {
      // 获取服务端返回的错误消息
      let message = res.message
      // 提示错误消息
      showMessage(message, 'error')
    }
  }).finally(() => formDialogRef.value.closeBtnLoading()) // 隐藏提交按钮 loading
}

// 删除标签
const deleteTagSubmit = (row) => {
  deleteTag(row.id).then((res) => {
    if (res.success === true) {
      showMessage('删除成功')
      // 重新请求分页接口，渲染数据
      getTableData()
    } else {
      // 获取服务端返回的错误消息
      let message = res.message
      // 提示错误消息
      showMessage(message, 'error')
    }
  })
}

// 标签输入框值
const inputValue = ref('')
// 已输入的标签数组
const dynamicTags = ref([])
// 标签输入框是否显示
const inputVisible = ref(false)
// 标签输入框的引用
const InputRef = ref('')

const handleClose = (tag) => {
  dynamicTags.value.splice(dynamicTags.value.indexOf(tag), 1)
}

const showInput = () => {
  inputVisible.value = true
  nextTick(() => {
    // 兼容 Element Plus 不同版本的 input 引用
    if (InputRef.value.input) {
      InputRef.value.input.focus()
    } else {
      InputRef.value.focus()
    }
  })
}

const handleInputConfirm = () => {
  if (inputValue.value) {
    if (!dynamicTags.value.includes(inputValue.value)) {
      dynamicTags.value.push(inputValue.value)
    } else {
      showMessage('标签已存在', 'warning')
    }
  }
  inputVisible.value = false
  inputValue.value = ''
}

</script>

<style scoped>
/* 覆盖表格悬停样式 */
:deep(.el-table--enable-row-hover .el-table__body tr:hover > td.el-table__cell) {
  background-color: #f8fafc;
}

/* 标签输入框样式微调 */
:deep(.el-input__wrapper) {
  box-shadow: none !important;
  padding: 0;
  background: transparent;
}

:deep(.el-input__inner) {
  height: 24px;
  line-height: 24px;
}
</style>