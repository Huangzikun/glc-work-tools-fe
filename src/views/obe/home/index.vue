<template>
  <div>
    <!-- 归档材料页面内容 -->
    <h1>归档材料</h1>
    <n-form :model="formData" ref="formRef">
      <n-form-item label="班级名字" path="class_name">
        <n-input v-model:value="formData.class_name" placeholder="请输入班级名字" />
      </n-form-item>
      <n-form-item label="课程名称" path="course_name">
        <n-input v-model:value="formData.course_name" placeholder="请输入课程名称" />
      </n-form-item>
      <n-form-item label="教师名称" path="teacher_name">
        <n-input v-model:value="formData.teacher_name" placeholder="请输入教师名称" />
      </n-form-item>
      <!-- 修改部分：动态添加目录列表 -->
      <n-form-item label="需要创建的目录" path="tags" :show-label="true">
        <n-dynamic-tags v-model:value="formData.need_mkdir_list" />
      </n-form-item>
      <n-form-item label="文件上传">
        <n-upload
          action="#"
          :default-file-list="[]"
          v-model:file-list="fileList"
          :multiple="false"
        >
          <n-button>选择文件</n-button>
        </n-upload>
      </n-form-item>
      <n-form-item>
        <n-button @click="submitForm">提交</n-button>
      </n-form-item>
    </n-form>
  </div>
</template>

<script lang="ts" setup>
import { ref } from 'vue';
import { useRouter } from 'vue-router'; // 假设使用 axios 进行请求，需提前安装
import { useMessage } from 'naive-ui';
import axios from 'axios';
import { getServiceBaseURL } from '@/utils/service';
const isHttpProxy = import.meta.env.DEV && import.meta.env.VITE_HTTP_PROXY === 'Y';

const router = useRouter();
const { otherBaseURL } = getServiceBaseURL(import.meta.env, isHttpProxy);

// 错误信息通过弹窗形式显示
const message = useMessage();

// 表单数据
const formData = ref({
  class_name: '',
  course_name: '',
  teacher_name: '',
  // 修改部分：使用数组存储需要创建的目录
  need_mkdir_list: []
});

// 表单引用
const formRef = ref(null);

// 文件列表
const fileList = ref([]);

const submitForm = async () => {
  try {
    // 构建请求数据，包含表单数据和文件
    const formDataToSend = new FormData();
    formDataToSend.append('class_name', formData.value.class_name);
    formDataToSend.append('course_name', formData.value.course_name);
    formDataToSend.append('teacher_name', formData.value.teacher_name);
    formDataToSend.append('need_mkdir_list', formData.value.need_mkdir_list);
    // 添加文件到表单数据
    fileList.value.forEach((fileInstance) => {
      formDataToSend.append('file', fileInstance.file); // 使用复数形式字段名
    });


    // 发送 POST 请求到后端接口
    const response = await axios.post('/obe/mkdir', formDataToSend, {
      baseURL: otherBaseURL['glc-work-tools'], // 使用 glc-work-tools 的 baseURL
      headers: {
        'Content-Type': 'multipart/form-data',
        'Access-Control-Allow-Origin': '*',
      }
    });

    console.log('请求成功:', response.data);
    if (response.data.code === 0) {
      // 新增跳转逻辑
      router.push({
        path: '/obe/filelist',
        query: {
          base_dir: encodeURIComponent(response.data.data.base_dir),
          directories: JSON.stringify(response.data.data.directory_list)
        }
      });
    } else {
      message.error(response.data.msg);
    }
  } catch (error) {
    console.log(error);
  }
};
</script>
