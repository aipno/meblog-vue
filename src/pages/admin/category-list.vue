<template>
  <div class="p-6">
    <!-- 顶部操作栏 -->
    <div class="flex flex-wrap items-center justify-between mb-6 gap-4">
      <!-- 左侧搜索区 -->
      <div class="flex items-center gap-3 flex-wrap">
        <div class="w-64">
          <el-input
              v-model="searchCategoryName"
              clearable
              placeholder="请输入分类名称"
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
        <el-button icon="Plus" type="primary" @click="addCategoryBtnClick">新增分类</el-button>
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
        <el-table-column label="分类名称" min-width="180" prop="name">
          <template #default="scope">
            <div class="flex items-center gap-2">
              <div class="w-8 h-8 rounded-lg bg-sky-50 text-sky-600 flex items-center justify-center">
                <el-icon>
                  <Folder/>
                </el-icon>
              </div>
              <span class="font-medium text-gray-700">{{ scope.row.name }}</span>
            </div>
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
            <el-popconfirm title="确定删除该分类吗？关联的文章将失去该分类。" @confirm="deleteCategorySubmit(scope.row)">
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

    <!-- 添加分类弹窗 -->
    <FormDialog ref="formDialogRef" destroyOnClose title="添加文章分类" width="400px" @submit="onSubmit">
      <el-form ref="formRef" :model="form" :rules="rules" label-position="top" size="large">
        <el-form-item label="分类名称" prop="name">
          <el-input
              v-model="form.name"
              clearable
              maxlength="20"
              placeholder="例如：后端技术、生活随笔..."
              prefix-icon="Folder"
              show-word-limit
          />
        </el-form-item>
      </el-form>
    </FormDialog>
  </div>
</template>

<script setup>
import {Clock, Folder} from '@element-plus/icons-vue'
import {addCategory, deleteCategory, getCategoryPageList} from '@/api/admin/category'
import {reactive, ref} from 'vue';
import moment from "moment";
import {showMessage} from '@/composables/utils'
import FormDialog from "@/components/FormDialog.vue"

// 分页查询的分类名称
const searchCategoryName = ref('')
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

// 重置查询条件
const reset = () => {
  searchCategoryName.value = ''
  pickDate.value = ''
  startDate.value = null
  endDate.value = null
  getTableData()
}

// 获取分页数据
function getTableData() {
  // 显示表格 loading
  tableLoading.value = true

  // 调用后台分页接口，并传入所需参数
  getCategoryPageList({
    current: current.value,
    size: size.value,
    startDate: startDate.value,
    endDate: endDate.value,
    name: searchCategoryName.value
  })
      .then((res) => {
        if (res.success === true) {
          tableData.value = res.data
          current.value = res.current
          size.value = res.size
          total.value = res.total
        }
      })
      .finally(() => {
        // 隐藏表格 loading
        tableLoading.value = false
      })
}

getTableData()

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
  name: ''
})

// 规则校验
const rules = {
  name: [
    {
      required: true,
      message: '分类名称不能为空',
      trigger: 'blur',
    },
    {min: 1, max: 20, message: '分类名称字数要求大于 1 个字符，小于 20 个字符', trigger: 'blur'},
  ]
}

const onSubmit = () => {
  // 先验证 form 表单字段
  formRef.value.validate((valid) => {
    if (!valid) {
      return false
    }

    formDialogRef.value.showBtnLoading()
    addCategory(form).then((res) => {
      if (res.success === true) {
        showMessage('添加成功')
        // 将表单中分类名称置空
        form.name = ''
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
    }).finally(() => formDialogRef.value.closeBtnLoading())
  })
}

const deleteCategorySubmit = (row) => {
  deleteCategory(row.id).then((res) => {
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
</script>

<style scoped>
/* 覆盖表格悬停样式，使其更柔和 */
:deep(.el-table--enable-row-hover .el-table__body tr:hover > td.el-table__cell) {
  background-color: #f8fafc;
}
</style>