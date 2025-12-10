<template>
  <!-- 设置语言为中文 -->
  <el-config-provider :locale="locale">
    <router-view></router-view>
    <!-- <Music></Music> -->
  </el-config-provider>
</template>

<script setup>
// import Music from "@/layouts/frontend/components/Music.vue";
import zhCn from 'element-plus/dist/locale/zh-cn.mjs';

// 设置 favicon
const setFavicon = () => {
  const favicon = import.meta.env.VITE_APP_FAVICON || '/src/assets/image/PwnnyBrigade.png';
  let link = document.querySelector("link[rel~='icon']");
  if (!link) {
    link = document.createElement('link');
    link.rel = 'icon';
    document.getElementsByTagName('head')[0].appendChild(link);
  }
  link.href = favicon;
};

setFavicon();

const locale = zhCn
</script>

<style>
#nprogress .bar {
  background: #409eff !important;
}

/* 暗黑模式 body 背景色 */
.dark body {
  --tw-bg-opacity: 1;
  background-color: rgb(17 24 39 / var(--tw-bg-opacity));
}
</style>