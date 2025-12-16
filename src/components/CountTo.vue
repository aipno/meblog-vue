<template>
  <!-- 数字滚动容器 -->
  <!-- 使用 tabular-nums 确保数字等宽，避免滚动时宽度抖动 -->
  <span :class="['tabular-nums', customClass]">
    {{ formattedValue }}
    <slot name="suffix">
      <span v-if="suffix" class="ml-1 text-sm font-normal opacity-70">{{ suffix }}</span>
    </slot>
  </span>
</template>

<script setup>
import { computed, onMounted, reactive, watch } from 'vue'
import gsap from 'gsap'

// 对外暴露的属性值
const props = defineProps({
  value: { // 目标数值
    type: Number,
    default: 0
  },
  initialValue: { // 初始数值
    type: Number,
    default: 0
  },
  duration: { // 动画时长 (秒)
    type: Number,
    default: 1.5
  },
  decimals: { // 小数位数
    type: Number,
    default: 0
  },
  suffix: { // 后缀单位
    type: String,
    default: ''
  },
  customClass: { // 自定义样式
    type: String,
    default: ''
  }
})

// 响应式数据
const d = reactive({
  num: props.initialValue
})

// 格式化后的数值
const formattedValue = computed(() => {
  return d.num.toFixed(props.decimals)
})

function animateToValue() {
  // 从当前值滚动到 value 属性指定的值
  gsap.to(d, {
    duration: props.duration,
    num: props.value,
    ease: "power2.out" // 添加缓动效果，先快后慢，更自然
  })
}

// 组件挂载后开始动画
onMounted(() => {
  animateToValue()
})

// 侦听属性, 监听 props.value 的变化
// 当数值更新时，自动从当前数值过渡到新数值
watch(() => props.value, () => animateToValue())
</script>

<style scoped>
/* 确保数字等宽，防止滚动时抖动 */
.tabular-nums {
  font-variant-numeric: tabular-nums;
}
</style>