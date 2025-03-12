<script setup lang="ts">
import { ref } from 'vue'
import axios from 'axios'

const inputText = ref('')
const translation = ref('')
const analysis = ref('')
const loading = ref(false)
const error = ref('')

const handleTranslate = async () => {
  if (!inputText.value.trim()) {
    error.value = '请输入要翻译的文本'
    return
  }

  loading.value = true
  error.value = ''

  try {
    const response = await axios.post('http://hk.klizz.asia:8000/api/translation', {
      text: inputText.value
    })

    // 打印返回的数据以进行调试
    console.log('Response data:', response.data)

    // 如果 response.data 是字符串且格式正确，直接解析
    const result = typeof response.data === 'string' ? JSON.parse(response.data) : response.data
    translation.value = result.translation
    analysis.value = result.analysis
  } catch (err) {
    if (err.response) {
      // 服务器返回了一个状态码，超出了2xx范围
      error.value = `请求失败: ${err.response.status} - ${err.response.data}`
    } else if (err.request) {
      // 请求已经发出，但没有收到响应
      error.value = '没有收到服务器响应，请检查网络连接'
    } else {
      // 其他错误
      error.value = `请求错误: ${err.message}`
    }
    console.error('Translation error:', err)
  } finally {
    loading.value = false
  }
}
</script>

<template>
  <div class="container">
    <div class="header">
      <h1>AI翻译助手</h1>
      <a href="https://klizz.online" target="_blank" class="profile-link">
        <span class="profile-icon">🤓</span>
        个人主页
      </a>
    </div>
    
    <div class="input-section">
      <textarea
        v-model="inputText"
        placeholder="请输入要翻译的文本..."
        rows="4"
        class="input-text"
      ></textarea>
      <button 
        @click="handleTranslate" 
        :disabled="loading"
        class="translate-btn"
      >
        {{ loading ? '翻译中...' : '翻译' }}
      </button>
    </div>

    <div v-if="error" class="error-message">
      {{ error }}
    </div>

    <div v-if="translation || analysis" class="result-section">
      <div class="result-box">
        <h3>翻译结果：</h3>
        <div class="result-content">{{ translation }}</div>
      </div>
      
      <div class="result-box">
        <h3>语法分析：</h3>
        <div class="result-content">{{ analysis }}</div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.container {
  max-width: 800px;
  margin: 0 auto;
  padding: 2rem;
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 2rem;
}

h1 {
  color: #2c3e50;
  margin: 0;
}

.profile-link {
  display: flex;
  align-items: center;
  text-decoration: none;
  color: #4CAF50;
  font-weight: 500;
  transition: color 0.3s;
}

.profile-link:hover {
  color: #45a049;
}

.profile-icon {
  margin-right: 0.5rem;
  font-size: 1.2rem;
}

.input-section {
  margin-bottom: 2rem;
}

.input-text {
  width: 100%;
  padding: 1rem;
  border: 2px solid #ddd;
  border-radius: 8px;
  font-size: 1rem;
  margin-bottom: 1rem;
  resize: vertical;
}

.translate-btn {
  background-color: #4CAF50;
  color: white;
  padding: 0.8rem 2rem;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 1rem;
  transition: background-color 0.3s;
}

.translate-btn:hover {
  background-color: #45a049;
}

.translate-btn:disabled {
  background-color: #cccccc;
  cursor: not-allowed;
}

.error-message {
  color: #ff4444;
  margin-bottom: 1rem;
  text-align: center;
}

.result-section {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 2rem;
  margin-top: 2rem;
}

.result-box {
  background-color: #f8f9fa;
  padding: 1.5rem;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.result-box h3 {
  color: #2c3e50;
  margin-bottom: 1rem;
}

.result-content {
  white-space: pre-wrap;
  line-height: 1.6;
  color: #333;
}
</style>