<template>
  <!-- PV 折线图容器 -->
  <!-- 增加 h-72 高度，与日历热点图保持一致，视觉更整齐 -->
  <div ref="chartContainer" class="w-full h-72 overflow-hidden"></div>
</template>

<script setup>
import * as echarts from 'echarts'
import { onBeforeUnmount, onMounted, ref, watch } from 'vue'
import { useDark, useResizeObserver } from '@vueuse/core'

// 对外暴露的属性值
const props = defineProps({
  value: { // 属性值名称
    type: Object, // 类型为对象
    default: () => ({ pvDates: [], pvCounts: [] })
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

// 初始化/更新折线图
function initLineChat() {
  if (!chartContainer.value) return

  if (!myChart) {
    myChart = echarts.init(chartContainer.value, null, { renderer: 'svg' });
  }

  // 从 props.value 中获取日期集合和 pv 访问量集合
  const pvDates = props.value?.pvDates || []
  const pvCounts = props.value?.pvCounts || []

  // 配色方案配置
  const isDarkMode = isDark.value
  const textColor = isDarkMode ? '#9ca3af' : '#6b7280' // text-gray-400 : text-gray-500
  const axisLineColor = isDarkMode ? '#374151' : '#e5e7eb' // border-gray-700 : border-gray-200
  const splitLineColor = isDarkMode ? '#1f2937' : '#f3f4f6' // border-gray-800 : border-gray-100
  const tooltipBgColor = isDarkMode ? 'rgba(31, 41, 55, 0.95)' : 'rgba(255, 255, 255, 0.95)'
  const tooltipBorderColor = isDarkMode ? '#4b5563' : '#e5e7eb'
  const tooltipTextColor = isDarkMode ? '#f3f4f6' : '#1f2937'

  // 主题色：Indigo
  const lineColor = '#6366f1' // indigo-500

  const option = {
    backgroundColor: 'transparent',
    tooltip: {
      trigger: 'axis',
      backgroundColor: tooltipBgColor,
      borderColor: tooltipBorderColor,
      textStyle: {
        color: tooltipTextColor,
        fontSize: 13
      },
      padding: [8, 12],
      extraCssText: 'box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06); border-radius: 8px;',
      axisPointer: {
        type: 'line',
        lineStyle: {
          color: lineColor,
          type: 'dashed'
        }
      },
      formatter: function (params) {
        const item = params[0];
        return `<div class="font-medium mb-1 text-xs text-gray-400">${item.axisValue}</div>
                  <div class="flex items-center gap-2">
                     <span class="w-2 h-2 rounded-full bg-indigo-500"></span>
                     <span class="text-sm font-bold">浏览量：${item.value}</span>
                  </div>`;
      }
    },
    grid: {
      top: '15%',
      left: '2%',
      right: '4%',
      bottom: '2%',
      containLabel: true
    },
    xAxis: {
      type: 'category',
      data: pvDates,
      boundaryGap: false, // 让折线从头画到尾，铺满
      axisLine: {
        show: false
      },
      axisTick: {
        show: false
      },
      axisLabel: {
        color: textColor,
        fontSize: 12,
        margin: 12
      }
    },
    yAxis: {
      type: 'value',
      splitLine: {
        lineStyle: {
          color: splitLineColor,
          type: 'dashed' // 虚线分割线，更轻量
        }
      },
      axisLabel: {
        color: textColor,
        fontSize: 12
      }
    },
    series: [
      {
        name: 'PV 访问量',
        data: pvCounts,
        type: 'line',
        smooth: true, // 平滑曲线
        showSymbol: false, // 默认不显示圆点，hover 时显示
        symbolSize: 8,
        itemStyle: {
          color: lineColor,
          borderWidth: 2,
          borderColor: '#fff'
        },
        lineStyle: {
          width: 3,
          color: lineColor,
          shadowColor: 'rgba(99, 102, 241, 0.3)', // 线条阴影
          shadowBlur: 10,
          shadowOffsetY: 5
        },
        areaStyle: {
          color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
            { offset: 0, color: 'rgba(99, 102, 241, 0.4)' }, // 顶部颜色
            { offset: 1, color: 'rgba(99, 102, 241, 0.01)' } // 底部透明
          ])
        }
      }
    ]
  };

  myChart.setOption(option);
}

// 监听数据变化
watch(() => props.value, () => initLineChat(), { deep: true })

// 监听暗黑模式变化，重绘图表
watch(isDark, () => {
  if (myChart) {
    myChart.dispose()
    myChart = null
    initLineChat()
  }
})

// 监听容器大小变化，实现自适应
useResizeObserver(chartContainer, () => {
  myChart && myChart.resize()
})

onMounted(() => {
  // 稍微延迟以确保容器尺寸计算正确
  setTimeout(() => initLineChat(), 50)
})
</script>

<style scoped></style>