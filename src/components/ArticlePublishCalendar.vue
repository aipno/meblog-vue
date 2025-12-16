<template>
  <!-- 日历热点图容器 -->
  <!-- 增加 h-72 高度，留出空间给图例 -->
  <div ref="chartContainer" class="w-full h-72 overflow-hidden"></div>
</template>

<script setup>
import * as echarts from 'echarts'
import { onBeforeUnmount, onMounted, ref, watch } from 'vue'
import { format, subMonths } from 'date-fns'
import { useDark, useResizeObserver } from '@vueuse/core'

// 对外暴露的属性值
const props = defineProps({
  value: { // 属性值名称
    type: Object, // 类型为对象
    default: () => ({})
  }
})

const chartContainer = ref(null)
let myChart = null
const isDark = useDark()

// 销毁实例
onBeforeUnmount(() => {
  if (myChart) {
    myChart.dispose()
    myChart = null
  }
})

// 初始化/更新日历热点图
function initCalendar() {
  if (!chartContainer.value) return

  // 如果为了适配暗黑模式切换，建议销毁重建或使用 setOption 更新配置
  if (!myChart) {
    myChart = echarts.init(chartContainer.value, null, { renderer: 'svg' });
  }

  // 当前日期
  const currentDate = new Date();
  // 半年前
  const sixMonthsAgo = subMonths(currentDate, 6)

  // 格式化后的开始、结束日期
  const startDate = format(sixMonthsAgo, 'yyyy-MM-dd')
  const endDate = format(currentDate, 'yyyy-MM-dd')

  // 日历热点数据
  const myData = []
  // 将传入的数据设置到 myData 数组中
  let map = props.value || {}
  for (let key in map) {
    myData.push([
      key,
      map[key]
    ]);
  }

  // 配色方案配置
  const isDarkMode = isDark.value
  const bgColor = 'transparent' // 背景透明，融入卡片
  const textColor = isDarkMode ? '#9ca3af' : '#6b7280' // text-gray-400 : text-gray-500
  const borderColor = isDarkMode ? '#1f2937' : '#fff' // 边框颜色作为间隔色

  // GitHub 风格配色
  const colorsLight = ['#ebedf0', '#9be9a8', '#40c463', '#30a14e', '#216e39']
  const colorsDark = ['#161b22', '#0e4429', '#006d32', '#26a641', '#39d353']
  const currentColors = isDarkMode ? colorsDark : colorsLight

  const option = {
    backgroundColor: bgColor,
    tooltip: {
      padding: 10,
      backgroundColor: isDarkMode ? '#374151' : '#fff',
      borderColor: isDarkMode ? '#4b5563' : '#e5e7eb',
      textStyle: {
        color: isDarkMode ? '#f3f4f6' : '#374151'
      },
      formatter: function (p) {
        const date = p.data[0];
        const count = p.data[1];
        return `<div class="text-sm">
                  <div class="mb-1 text-gray-400">${date}</div>
                  <div>发布了 <span class="font-bold text-sky-500">${count}</span> 篇文章</div>
                </div>`;
      }
    },
    visualMap: {
      show: true,
      min: 0,
      max: 4, // 阈值调整，根据实际发布频率优化
      calculable: false,
      type: 'piecewise',
      orient: 'horizontal',
      left: 'center',
      bottom: 0,
      itemWidth: 10,
      itemHeight: 10,
      itemGap: 4,
      textStyle: {
        color: textColor,
        fontSize: 12
      },
      pieces: [
        { min: 4, label: 'More', color: currentColors[4] },
        { min: 3, max: 3, label: '', color: currentColors[3] },
        { min: 2, max: 2, label: '', color: currentColors[2] },
        { min: 1, max: 1, label: 'Less', color: currentColors[1] },
        { max: 0, label: '', color: currentColors[0] }
      ]
    },
    calendar: {
      top: 25,
      left: 30,
      right: 30,
      cellSize: ['auto', 16], // 宽度自适应，高度固定
      range: [startDate, endDate],
      itemStyle: {
        color: currentColors[0],
        borderColor: borderColor, // 使用边框颜色模拟间距
        borderWidth: 3,
        borderRadius: 2 // 小圆角
      },
      splitLine: { show: false },
      yearLabel: { show: false },
      monthLabel: {
        nameMap: 'cn',
        color: textColor,
        fontSize: 12,
        margin: 10
      },
      dayLabel: {
        firstDay: 1,
        nameMap: ['日', '一', '', '三', '', '五', ''],
        color: textColor,
        fontSize: 12
      }
    },
    series: {
      type: 'heatmap',
      coordinateSystem: 'calendar',
      data: myData,
      itemStyle: {
        borderRadius: 3
      }
    }
  };

  myChart.setOption(option);
}

// 监听数据变化
watch(() => props.value, () => initCalendar())

// 监听暗黑模式变化，重绘图表以更新颜色
watch(isDark, () => {
  if (myChart) {
    myChart.dispose()
    myChart = null
    initCalendar()
  }
})

// 监听容器大小变化，实现自适应
useResizeObserver(chartContainer, () => {
  myChart && myChart.resize()
})

onMounted(() => {
  // 稍微延迟以确保容器尺寸计算正确
  setTimeout(() => initCalendar(), 50)
})
</script>

<style scoped></style>