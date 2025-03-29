<script setup>
import { useRoute } from 'vue-router';

const route = useRoute();
const baseDir = decodeURIComponent(route.query.base_dir);
const directories = JSON.parse(route.query.directories);
</script>

<template>
  <div>
    <h1>生成的文件目录列表</h1>
    <p>基础路径：{{ baseDir }}</p>
    <n-list>
      <n-list-item v-for="(dir, index) in directories" :key="index">
        {{ dir }}
      </n-list-item>
    </n-list>
  </div>
</template>
