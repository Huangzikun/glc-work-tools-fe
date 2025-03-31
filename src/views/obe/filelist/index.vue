<template>
  <div>
    <n-form :model="{ path: formPath }" style="max-width: 600px; margin-bottom: 24px">
      <n-form-item label="基础路径">
        <n-input
          v-model:value="formPath"
          placeholder="请输入/选择基础路径"
          :disabled="loading"
        />
      </n-form-item>
      <n-button
        type="primary"
        :loading="loading"
        @click="handleSubmit"
      >查询</n-button>
    </n-form>

    <template v-if="directories.length">
      <n-h1>生成的文件目录列表</n-h1>
      <n-p>当前路径：{{ baseDir }}</n-p>
      <n-list>
        <n-list-item v-for="(dir, index) in directories" :key="index">
          <div class="flex justify-between items-center">
            <div>
              <n-text strong>{{ dir.directory }}</n-text>
              <n-text depth="3" class="ml-4">包含 {{ dir.file_count }} 个文件</n-text>
            </div>
            <n-button
              type="primary"
              size="small"
              @click="
                router.push({
                  path: '/obe/uploadfile',
                  query: { filepath: encodeURIComponent(baseDir + '/' + dir.directory) }
                })
              "
            >
              上传文件
            </n-button>
          </div>
        </n-list-item>
      </n-list>
    </template>
  </div>
</template>

<script setup>
import { onMounted, ref } from 'vue';
import { useRoute, useRouter } from 'vue-router';
import { NButton, NForm, NFormItem, NInput, useMessage } from 'naive-ui';
import axios from 'axios';
import { getServiceBaseURL } from '@/utils/service.js';

const message = useMessage();
const route = useRoute();
const router = useRouter();
const baseDir = ref(route.query.base_dir ? decodeURIComponent(route.query.base_dir) : '');
const directories = ref([]); // 移除路由参数初始化逻辑
const formPath = ref(baseDir.value); // 默认使用当前baseDir
const loading = ref(false);
const isHttpProxy = import.meta.env.DEV && import.meta.env.VITE_HTTP_PROXY === 'Y';
const { otherBaseURL } = getServiceBaseURL(import.meta.env, isHttpProxy);

// 添加挂载时自动查询逻辑
onMounted(() => {
  if (route.query.base_dir) {
    formPath.value = baseDir.value;
    handleSubmit();
  }
});

async function handleSubmit() {
  if (!formPath.value.trim()) {
    message.error('请输入有效路径');
    return;
  }

  try {
    loading.value = true;

    const formDataToSend = new FormData();
    formDataToSend.append('path', formPath.value);
    // 发送 POST 请求到后端接口
    const response = await axios.post('/obe/filelist', formDataToSend, {
      baseURL: otherBaseURL['glc-work-tools'], // 使用 glc-work-tools 的 baseURL
      headers: {
        'Content-Type': 'multipart/form-data',
        'Access-Control-Allow-Origin': '*',
      }
    });

    if (response.data.code === 0) {
      // 更新数据
      baseDir.value = response.data.data.base_dir;
      directories.value = response.data.data.directory_list; // 直接使用目录对象数组
      message.success(response.data.msg);
    } else {
      message.error(response.data.msg);
    }
  } catch (error) {
    // 错误处理
    const errorMsg = error.response?.data?.msg || error.message || '请求失败';
    message.error(errorMsg);
  } finally {
    loading.value = false;
  }
}
</script>
