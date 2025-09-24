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
const selectedStyle = ref<string>('爆款小红书')
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
  const STYLE_PROMPTS = {
    爆款小红书: `
你是一个小红书爆款标题生成器，请根据主题“{主题}”，生成5个吸引眼球、带emoji、口语化、有悬念或情绪共鸣的标题。

要求：
- 每个标题不超过20字
- 必须带1-2个emoji
- 用“我”、“你”、“谁懂啊”、“救命”、“真的绝了”等口语词
- 风格：活泼、种草、情绪共鸣

示例：
🔥谁懂啊！健身3个月，腰围狂减8cm！
💥打工人必看！5分钟搞定周报，老板狂夸！
✨素人改造｜换发型=换头！闺蜜追着问链接！

请直接输出标题，不要解释，不要序号，每行一个。
⚠️ 注意：不要解释词语含义！不要写“示例”！直接生成标题！
`,

    知乎专业风: `
你是一个知乎高赞标题生成器，请根据主题“{主题}”，生成5个理性、有深度、带方法论或数据支撑的标题。

要求：
- 每个标题不超过25字
- 用“为什么”、“如何”、“有哪些”、“深度解析”等词
- 风格：专业、冷静、有信息增量

示例：
为什么90%的人健身3个月就放弃？科学解析+解决方案
如何用「番茄工作法」提升300%效率？亲测有效
有哪些不为人知的租房避坑指南？律师朋友告诉我这些

请直接输出标题，不要解释，不要序号，每行一个。
⚠️ 注意：不要解释词语含义！不要写“示例”！直接生成标题！
`,

    抖音短平快: `
你是一个抖音爆款标题生成器，请根据主题“{主题}”，生成5个前3秒就能抓住眼球的标题。

要求：
- 每个标题不超过15字
- 开头必须有强钩子：“注意！”、“速看！”、“别划走！”、“最后1秒惊呆！”
- 用感叹号、问号、省略号制造悬念
- 风格：快节奏、强冲击、反转结局

示例：
注意！这样睡觉=慢性自杀！
速看！月薪3千到3万，我只做了这件事...
别划走！99%人不知道的微信隐藏功能！

请直接输出标题，不要解释，不要序号，每行一个。
⚠️ 注意：不要解释词语含义！不要写“示例”！直接生成标题！
`,

    毒舌吐槽风: `
你是一个毒舌段子手，请根据主题“{主题}”，生成5个带反转、情绪、吐槽的标题。

要求：
- 每个标题不超过20字
- 用“笑死”、“谁懂”、“离谱”、“求你们别...”、“我又...”等情绪词
- 带反转 or 自嘲 or 夸张
- 风格：犀利、幽默、有网感

示例：
笑死！谁家好人上班带饭啊？
谁懂啊！男朋友说“多喝热水”那一刻我裂开了
求你们别再买网红小家电了！智商税第一名！

请直接输出标题，不要解释，不要序号，每行一个。
⚠️ 注意：不要解释词语含义！不要写“示例”！直接生成标题！
`,
  }

  // ✅ 构建 Prompt 的函数
  // 修改 buildPrompt 函数签名，使用联合类型限定 style 参数
  const buildPrompt = (promptText: string, style: string) => {
    const template =
      STYLE_PROMPTS[style as keyof typeof STYLE_PROMPTS] || STYLE_PROMPTS['爆款小红书']
    return template.replace('{主题}', promptText.trim())
  }

  // const API_BASE =
  //   import.meta.env.MODE === 'development'
  //     ? 'http://localhost:3001'
  //     : 'https://title-generator-backend-production.up.railway.app' // ✅ 线上地址！
  // const API_BASE = 'https://title-generator-backend-production-23e8.up.railway.app'
  const API_BASE = 'https://1380218698-e0n8afp793.ap-guangzhou.tencentscf.com'
  try {
    const response = await fetch(`${API_BASE}`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        prompt: buildPrompt(topic.value.trim(), selectedStyle.value),
        // style: selectedStyle.value,
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
    // eslint-disable-next-line @typescript-eslint/no-unused-vars
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
    // eslint-disable-next-line @typescript-eslint/no-unused-vars
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
