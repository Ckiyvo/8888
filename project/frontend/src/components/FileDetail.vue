<template>
  <div class="file-detail-container">
    <!-- 路径栏 -->
    <div class="path-bar">
      <router-link :to="{ path: `/project/${projectName}` }"> / {{ projectName }}</router-link>
      / {{ filename }}
    </div>
    <!-- 导航按钮 -->
    <div class="navigation-buttons">
      <button :disabled="isFirstFile" @click="prevFile" class="action-button">上一个</button>
      <button :disabled="isLastFile" @click="nextFile" class="action-button">下一个</button>
    </div>
    <!-- 灰色细线 -->
    <div class="path-bar-divider"></div>

    <h2>{{ filename }} 文件详情</h2>
    <p>这是 {{ filename }} 文件的详细信息页面。</p>
    <!-- 文件预览区域 -->
    <div v-if="previewUrl">
      <div v-if="isTextFile">
        <pre>{{ textContent }}</pre>
      </div>
      <div v-else-if="isImageFile">
        <img :src="previewUrl" alt="预览图" style="max-width: 100%;">
      </div>
      <div v-else-if="isAudioFile">
        <audio controls :src="previewUrl"></audio>
      </div>
      <div v-else-if="isVideoFile">
        <video controls :src="previewUrl"></video>
      </div>
      <div v-else>
        <a :href="previewUrl" download>下载文件</a>
      </div>
    </div>
  </div>
</template>

<script>
import { useRoute, useRouter } from 'vue-router';
import axios from 'axios';
import { ref, onMounted } from 'vue';

export default {
  name: 'FileDetail',
  setup() {
    const route = useRoute();
    const router = useRouter();
    const projectName = route.params.projectname;
    const filename = route.params.filename;
    const currentIndex = parseInt(route.query.index);
    const project = ref(null);
    const previewUrl = ref('');
    const textContent = ref('');
    const isTextFile = ref(false);
    const isImageFile = ref(false);
    const isAudioFile = ref(false);
    const isVideoFile = ref(false);

    const prevFile = () => {
      if (currentIndex > 0) {
        const prevIndex = currentIndex - 1;
        axios.get(`http://127.0.0.1:8000/api/get_projects/`)
          .then((response) => {
            const projects = response.data.projects;
            project.value = projects.find(p => p.name === projectName);
            if (project.value) {
              const prevFile = project.value.files[prevIndex];
              const path = `/project/${projectName}/${prevFile.name}`;
              router.push({ path, query: { index: prevIndex } });
            }
          })
          .catch((error) => {
            console.error('获取项目文件列表失败:', error);
          });
      }
    };

    const nextFile = () => {
      axios.get(`http://127.0.0.1:8000/api/get_projects/`)
        .then((response) => {
          const projects = response.data.projects;
          project.value = projects.find(p => p.name === projectName);
          if (project.value) {
            const nextIndex = currentIndex + 1;
            if (project.value.files && nextIndex < project.value.files.length) {
              const nextFile = project.value.files[nextIndex];
              const path = `/project/${projectName}/${nextFile.name}`;
              router.push({ path, query: { index: nextIndex } });
            }
          }
        })
        .catch((error) => {
          console.error('获取项目文件列表失败:', error);
        });
    };

    const isFirstFile = currentIndex === 0;
    const isLastFile = project.value && project.value.files && currentIndex === project.value.files.length - 1;

    onMounted(() => {
      const fileExtension = filename.split('.').pop().toLowerCase();
      isTextFile.value = ['.txt', '.csv'].includes(`.${fileExtension}`);
      isImageFile.value = ['.jpg', '.jpeg', '.png'].includes(`.${fileExtension}`);
      isAudioFile.value = ['.wav', '.mp3'].includes(`.${fileExtension}`);
      isVideoFile.value = ['.aac', '.mp4'].includes(`.${fileExtension}`);

      axios.get(`http://127.0.0.1:8000/api/get_file_content/${projectName}/${filename}/`, { responseType: 'blob' })
        .then((response) => {
          const url = URL.createObjectURL(response.data);
          previewUrl.value = url;

          if (isTextFile.value) {
            const reader = new FileReader();
            reader.onload = () => {
              textContent.value = reader.result;
            };
            reader.readAsText(response.data);
          }
        })
        .catch((error) => {
          console.error('获取文件内容失败:', error);
        });
    });

    return {
      projectName,
      filename,
      prevFile,
      nextFile,
      isFirstFile,
      isLastFile,
      previewUrl,
      textContent,
      isTextFile,
      isImageFile,
      isAudioFile,
      isVideoFile
    };
  }
};
</script>

<style scoped>
.file-detail-container {
  padding: 0px;
}

/* 路径栏及导航按钮容器样式 */
.path-bar-container {
  margin-bottom:10px;
  width: 100%; /* 确保容器宽度占满 */
}

/* 路径栏样式 */
.path-bar {
  font-size: 14px;
  color: #666;
  display: inline-block;
}

.path-bar a {
  color: #007bff;
  text-decoration: none; /* 取消下划线 */
}

.path-bar a:hover {
  text-decoration: underline; /* 鼠标悬停时显示下划线 */
}

/* 导航按钮样式 */
.navigation-buttons {
  float: right;
  
}

.action-button {
  padding: 5px 10px;
  margin-right: 5px;
  border: none;
  background-color: #007bff;
  color: white;
  border-radius: 3px;
  cursor: pointer;
}

.action-button:hover {
  background-color: #0056b3;
}

/* 清除浮动 */
.clearfix::after {
  content: "";
  display: table;
  clear: both;
}
/* 路径栏分隔线 */
.path-bar-divider {
  border-bottom: 1px solid #ccc;
  margin-bottom: 10px;
  margin-top: 20px;
  width: 100%;
}
</style>