<template>
  <div class="max-w-2xl mx-auto p-6">
    <h1
      class="text-3xl font-bold text-center mb-6 bg-gradient-to-r from-pink-500 to-purple-600 text-white py-3 rounded-lg"
    >
      🎯 AI 爆款标题生成器
    </h1>

    <!-- 主题输入 -->
    <div class="mb-4">
      <label class="block text-gray-700 font-medium mb-2">📝 输入你的内容主题：</label>
      <input
        v-model="topic"
        type="text"
        placeholder="比如：显瘦穿搭、职场沟通、早餐食谱..."
        class="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-purple-400 outline-none"
        @keypress.enter="generateTitles"
      />
    </div>

    <!-- 风格选择 -->
    <div class="mb-4">
      <label class="block text-gray-700 font-medium mb-2">🎨 选择风格：</label>
      <select
        v-model="selectedStyle"
        class="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-purple-400 outline-none"
      >
        <option value="爆款小红书">💄 爆款小红书风（带emoji+悬念）</option>
        <option value="知乎专业风">🧠 知乎专业风（理性+方法论）</option>
        <option value="抖音短平快">📱 抖音短平快（前3秒抓眼球）</option>
        <option value="毒舌吐槽风">🙄 毒舌吐槽风（反转+情绪）</option>
      </select>
    </div>

    <!-- 按钮组 -->
    <div class="flex gap-3 mb-6">
      <button
        @click="generateTitles"
        :disabled="isLoading"
        class="flex-1 bg-gradient-to-r from-purple-500 to-pink-500 text-white py-3 px-6 rounded-lg font-bold hover:from-pink-500 hover:to-purple-500 transition shadow-md disabled:opacity-70"
      >
        <span v-if="!isLoading">🚀 生成标题</span>
        <span v-else>⏳ 生成中...</span>
      </button>
      <button
        @click="copyAll"
        :disabled="titles.length === 0"
        class="flex-1 bg-gray-200 text-gray-800 py-3 px-6 rounded-lg font-medium hover:bg-gray-300 transition shadow disabled:opacity-50"
      >
        📋 复制全部
      </button>
    </div>

    <!-- Loading 状态 -->
    <div v-if="isLoading" class="text-center py-6">
      <div
        class="inline-block animate-spin rounded-full h-8 w-8 border-t-2 border-b-2 border-purple-500"
      ></div>
      <p class="mt-2 text-gray-600">AI 正在疯狂创作中... 💪</p>
    </div>

    <!-- 结果区域 -->
    <div v-if="titles.length > 0" class="space-y-3">
      <div
        v-for="(title, index) in titles"
        :key="index"
        class="bg-white p-4 rounded-xl shadow-sm border border-gray-100 hover:shadow-md transition group relative"
      >
        <div class="flex items-start justify-between">
          <p class="text-gray-800 leading-relaxed flex-1">{{ title }}</p>
          <button
            @click="copySingle(title)"
            class="ml-3 text-purple-500 hover:text-purple-700 opacity-0 group-hover:opacity-100 transition-opacity text-sm font-medium absolute right-4 top-4"
            title="复制"
          >
            复制
          </button>
        </div>
      </div>
    </div>

    <!-- 错误提示 -->
    <div v-if="error" class="mt-4 p-3 bg-red-100 text-red-700 rounded-lg">❌ {{ error }}</div>
    <!-- Toast 提示 -->
    <transition name="fade">
      <div
        v-if="toast.show"
        :class="[
          'fixed bottom-6 left-1/2 transform -translate-x-1/2 px-6 py-3 rounded-xl text-white font-medium shadow-lg transition-all duration-300 z-50',
          toast.type === 'success'
            ? 'bg-gradient-to-r from-green-500 to-emerald-500'
            : 'bg-gradient-to-r from-red-500 to-pink-500',
        ]"
        style="min-width: 280px; max-width: 90vw"
      >
        {{ toast.message }}
      </div>
    </transition>
  </div>
</template>

<script lang="ts" setup>
import { ref } from 'vue'

// 类型定义
interface GenerateResponse {
  titles: string[]
  error?: string
}

// 响应式数据
const topic = ref('')
const selectedStyle = ref('爆款小红书')
const titles = ref<string[]>([])
const isLoading = ref(false)
const error = ref<string | null>(null)
// 在 <script setup> 顶部，其他 ref 旁边添加：

const toast = ref({
  show: false,
  message: '',
  type: 'success', // 'success' | 'error'
})

// 显示 Toast 的函数
const showToast = (message: string, type: 'success' | 'error' = 'success') => {
  toast.value = { show: true, message, type }
  setTimeout(() => {
    toast.value.show = false
  }, 3000)
}
// 生成标题
const generateTitles = async () => {
  if (!topic.value.trim()) {
    error.value = '请输入主题内容！'
    return
  }

  isLoading.value = true
  error.value = null
  titles.value = []
  const API_BASE =
    import.meta.env.MODE === 'development'
      ? 'http://localhost:3001'
      : 'https://title-generator-backend-production.up.railway.app' // ✅ 线上地址！
  try {
    const response = await fetch('https://title-generator-backend-production.up.railway.app', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        prompt: topic.value.trim(),
        style: selectedStyle.value,
        // body: JSON.stringify({ theme: theme.value }), // 👈 这行是关键！
      }),
    })

    if (!response.ok) {
      throw new Error('网络请求失败')
    }

    const data: GenerateResponse = await response.json()

    if (data.error) {
      error.value = data.error
      return
    }

    titles.value = data.titles || []
  } catch (err) {
    error.value = '生成失败，请重试'
    console.error(err)
  } finally {
    isLoading.value = false
  }
}

const copySingle = async (text: string) => {
  try {
    await navigator.clipboard.writeText(text)
    showToast(`✅ 已复制`, 'success')
  } catch (err) {
    showToast('❌ 复制失败，请手动选择复制', 'error')
  }
}

const copyAll = async () => {
  if (titles.value.length === 0) {
    showToast('⚠️ 请先生成标题', 'error')
    return
  }

  try {
    await navigator.clipboard.writeText(titles.value.join('\n\n'))
    showToast('✅ 已复制全部标题！', 'success')
  } catch (err) {
    showToast('❌ 复制失败，请手动选择复制', 'error')
  }
}
</script>

<style scoped>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
