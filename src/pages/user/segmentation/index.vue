<template>
  <div class="min-h-screen bg-base-100 p-4">
    <div class="max-w-7xl mx-auto">
      <!-- 页面标题 -->
      <RevealMotion>
        <div class="text-center mb-8">          <h1 class="text-5xl font-bold text-primary mb-4">
              AI 道路缺陷检测系统
            </h1>
            <p class="text-xl text-base-content/70">智能道路图像分析 · 精准缺陷识别</p>
        </div>
      </RevealMotion>

      <!-- 主要内容区域 -->
      <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
        <!-- 左侧：图片上传区域 -->
        <div class="lg:col-span-1">
          <RevealMotion :delay="0.1">
            <div class="card bg-base-200/80 backdrop-blur-sm rounded-2xl shadow-xl border border-base-300 p-6">
              <h2 class="text-2xl font-bold text-base-content mb-6 flex items-center">
                <span class="text-3xl mr-3"></span>
                图片上传
              </h2>
              
              <!-- 拖拽上传区域 -->
              <div 
                class="border-2 border-dashed rounded-xl p-6 text-center transition-all duration-300 hover:scale-105"
                :class="{
                  'border-primary bg-primary/10 shadow-lg': isDragOver,
                  'border-error bg-error/10': uploadError,
                  'border-base-300 hover:border-primary hover:bg-primary/5': !isDragOver && !uploadError
                }"
                @dragover.prevent="isDragOver = true"
                @dragleave.prevent="isDragOver = false"
                @drop="handleDrop"
              >
                <div v-if="!selectedFile" class="space-y-4">
                  <div class="text-6xl">
                    🎯
                  </div>
                  <div>
                    <p class="text-lg font-medium text-base-content mb-3">拖拽道路图像到此处</p>
                    <button 
                      class="btn btn-primary px-6 py-3 rounded-xl font-medium transition-all duration-300 transform hover:scale-105 shadow-lg"
                      @click="fileInput?.click()"
                    >
                      📁 选择文件
                    </button>
                  </div>
                  <p class="text-sm text-base-content/60">支持 JPG、PNG 格式 · 最大 20MB</p>
                </div>
                
                <!-- 文件预览 -->
                <div v-else class="space-y-4">
                  <div class="relative inline-block">
                    <img 
                      :src="previewUrl" 
                      alt="预览图片" 
                      class="max-w-full max-h-48 rounded-xl shadow-lg"
                    >
                    <button 
                      class="btn btn-sm btn-circle btn-error absolute -top-2 -right-2 transition-colors"
                      @click="clearFile"
                    >
                      ✕
                    </button>
                  </div>
                  <div class="text-sm text-base-content/70 bg-base-200 rounded-lg p-3">
                    <p class="font-medium">📄 {{ selectedFile.name }}</p>
                    <p class="text-base-content/60">📊 {{ formatFileSize(selectedFile.size) }}</p>
                  </div>
                </div>
              </div>
              
              <!-- 错误提示 -->
              <div v-if="uploadError" class="alert alert-error mt-4 p-4 rounded-xl">
                <span class="flex items-center">
                  <span class="text-xl mr-2">⚠️</span>
                  {{ uploadError }}
                </span>
              </div>
              
              <!-- 隐藏的文件输入 -->
              <input 
                ref="fileInput"
                type="file" 
                accept="image/*" 
                class="hidden" 
                @change="handleFileSelect"
              >
              
              <!-- 开始分割按钮 -->
              <div class="mt-6">
                <button 
                  class="w-full py-4 rounded-xl font-bold text-lg transition-all duration-300 transform hover:scale-105 shadow-lg"
                  :class="{
                    'btn-success': selectedFile && !isLoading,
                    'btn-disabled': !selectedFile || isLoading,
                    'animate-pulse': isLoading
                  }"
                  :disabled="!selectedFile || isLoading"
                  @click="performSegmentation"
                >
                  <span v-if="!isLoading" class="flex items-center justify-center">
                    <span class="text-2xl mr-2">🚀</span>
                    开始AI检测
                  </span>
                  <span v-else class="flex items-center justify-center">
                    <span class="text-2xl mr-2 animate-spin">⚡</span>
                    AI检测中...
                  </span>
                </button>
              </div>
            </div>
          </RevealMotion>
        </div>

        <!-- 右侧：历史记录区域 -->
        <div class="lg:col-span-2">
          <RevealMotion :delay="0.2">
            <div class="card bg-base-200/80 backdrop-blur-sm rounded-2xl shadow-xl border border-base-300 p-6">
              <div class="flex items-center justify-between mb-6">
                <h2 class="text-2xl font-bold text-base-content flex items-center">
                  <span class="text-3xl mr-3">📊</span>
                  检测历史
                </h2>
                <div class="flex items-center space-x-2">
                  <span class="text-sm text-base-content/60">共 {{ historyRecords.length }} 条记录</span>
                  <button 
                    v-if="historyRecords.length > 0"
                    class="btn btn-ghost btn-sm text-error hover:text-error-focus text-sm underline"
                    @click="clearHistory"
                  >
                    清空历史
                  </button>
                </div>
              </div>
              
              <!-- 历史记录网格 -->
              <div v-if="historyRecords.length > 0" class="grid grid-cols-1 md:grid-cols-2 xl:grid-cols-3 gap-4 max-h-96 overflow-y-auto">
                <div 
                  v-for="(record, index) in historyRecords" 
                  :key="index"
                  class="card bg-base-100 rounded-xl p-4 border border-base-300 hover:shadow-lg transition-all duration-300 cursor-pointer transform hover:scale-105"
                  @click="openResultModal(record)"
                >
                  <div class="relative mb-3">
                    <img 
                      :src="getImageUrl(record.original_image_path)" 
                      alt="历史记录" 
                      class="w-full h-32 object-cover rounded-lg"
                    >
                    <div class="absolute top-2 right-2 bg-black/70 text-white text-xs px-2 py-1 rounded-full">
                      {{ formatDate(record.timestamp) }}
                    </div>
                  </div>
                  
                  <div class="space-y-2">
                    <div class="flex justify-between items-center">
                      <span class="text-sm font-medium text-base-content/80">缺陷置信度</span>
                    <span class="text-sm font-bold text-secondary">{{ (record.confidence * 100).toFixed(1) }}%</span>
                    </div>
                    <div class="flex justify-between items-center">
                      <span class="text-sm font-medium text-base-content/80">缺陷比例</span>
                    <span class="text-sm font-bold text-primary">{{ (record.tumor_ratio * 100).toFixed(2) }}%</span>
                    </div>
                    <div class="text-xs text-base-content/60 truncate">
                      {{ record.original_filename || '未知文件' }}
                    </div>
                  </div>
                </div>
              </div>
              
              <!-- 空状态 -->
              <div v-else class="text-center py-12">
                <div class="text-6xl mb-4">📈</div>
                <p class="text-base-content/60 text-lg">暂无检测记录</p>
                <p class="text-base-content/50 text-sm mt-2">上传道路图像开始您的第一次AI检测</p>
              </div>
            </div>
          </RevealMotion>
        </div>
      </div>
    </div>

    <!-- 结果详情弹窗 -->
    <div v-if="showResultModal" class="modal modal-open fixed inset-0 bg-black/50 flex items-center justify-center z-50 p-4">
      <div class="modal-box bg-base-100 rounded-2xl max-w-6xl w-full max-h-[90vh] overflow-y-auto">
        <div class="p-6">
          <!-- 弹窗标题 -->
          <div class="flex items-center justify-between mb-6">
            <h3 class="text-2xl font-bold text-base-content flex items-center">
                  <span class="text-3xl mr-3">🎯</span>
                  检测结果详情
                </h3>
            <button 
              class="btn btn-sm btn-circle btn-ghost transition-colors"
              @click="closeResultModal"
            >
              ✕
            </button>
          </div>
          
          <!-- 统计信息 -->
          <div class="grid grid-cols-2 md:grid-cols-4 gap-4 mb-8">
            <div class="bg-secondary/10 rounded-xl p-4 text-center border border-secondary/20">
              <div class="text-2xl mb-2">🎯</div>
              <div class="text-sm text-base-content/70 mb-1">缺陷置信度</div>
              <div class="text-2xl font-bold text-secondary">{{ (currentResult.confidence * 100).toFixed(1) }}%</div>
            </div>
            <div class="bg-primary/10 rounded-xl p-4 text-center border border-primary/20">
              <div class="text-2xl mb-2">📏</div>
              <div class="text-sm text-base-content/70 mb-1">缺陷面积</div>
              <div class="text-xl font-bold text-primary">{{ currentResult.tumor_area?.toLocaleString() }}</div>
              <div class="text-xs text-base-content/60">像素</div>
            </div>
            <div class="bg-success/10 rounded-xl p-4 text-center border border-success/20">
              <div class="text-2xl mb-2">📊</div>
              <div class="text-sm text-base-content/70 mb-1">总面积</div>
              <div class="text-xl font-bold text-success">{{ currentResult.total_area?.toLocaleString() }}</div>
              <div class="text-xs text-base-content/60">像素</div>
            </div>
            <div class="bg-warning/10 rounded-xl p-4 text-center border border-warning/20">
              <div class="text-2xl mb-2">📈</div>
              <div class="text-sm text-base-content/70 mb-1">缺陷比例</div>
              <div class="text-xl font-bold text-warning">{{ (currentResult.tumor_ratio * 100).toFixed(2) }}%</div>
            </div>
          </div>
          
          <!-- 图片展示 -->
          <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
            <!-- 原始图片 -->
            <div class="space-y-3">
              <h4 class="font-bold text-lg text-base-content flex items-center">
                <span class="text-xl mr-2">🖼️</span>
                原始图片
              </h4>
              <div class="relative group">
                <img 
                  :src="getImageUrl(currentResult.original_image_path)"
                  alt="原始图片"
                  class="w-full rounded-xl shadow-lg cursor-pointer transition-transform group-hover:scale-105"
                  @click="openImageModal(getImageUrl(currentResult.original_image_path), '原始图片')"
                >
                <div class="absolute inset-0 bg-black/0 group-hover:bg-black/20 transition-colors rounded-xl flex items-center justify-center">
                  <span class="text-white opacity-0 group-hover:opacity-100 transition-opacity font-medium">🔍 点击放大</span>
                </div>
              </div>
            </div>
            
            <!-- 分割掩码 -->
            <div class="space-y-3">
              <h4 class="font-bold text-lg text-base-content flex items-center">
                <span class="text-xl mr-2">🎭</span>
                分割掩码
              </h4>
              <div class="relative group">
                <img 
                  :src="getImageUrl(currentResult.mask_image_path)"
                  alt="分割掩码"
                  class="w-full rounded-xl shadow-lg cursor-pointer transition-transform group-hover:scale-105"
                  @click="openImageModal(getImageUrl(currentResult.mask_image_path), '分割掩码')"
                >
                <div class="absolute inset-0 bg-black/0 group-hover:bg-black/20 transition-colors rounded-xl flex items-center justify-center">
                  <span class="text-white opacity-0 group-hover:opacity-100 transition-opacity font-medium">🔍 点击放大</span>
                </div>
              </div>
            </div>
            
            <!-- 叠加结果 -->
            <div class="space-y-3">
              <h4 class="font-bold text-lg text-base-content flex items-center">
                <span class="text-xl mr-2">🎨</span>
                叠加结果
              </h4>
              <div class="relative group">
                <img 
                  :src="getImageUrl(currentResult.overlay_image_path)"
                  alt="叠加结果"
                  class="w-full rounded-xl shadow-lg cursor-pointer transition-transform group-hover:scale-105"
                  @click="openImageModal(getImageUrl(currentResult.overlay_image_path), '叠加结果')"
                >
                <div class="absolute inset-0 bg-black/0 group-hover:bg-black/20 transition-colors rounded-xl flex items-center justify-center">
                  <span class="text-white opacity-0 group-hover:opacity-100 transition-opacity font-medium">🔍 点击放大</span>
                </div>
              </div>
            </div>
          </div>
          
          <!-- 操作按钮 -->
          <div class="flex justify-center space-x-4 mt-8">
            <button 
              class="px-6 py-3 bg-gradient-to-r from-blue-500 to-blue-600 hover:from-blue-600 hover:to-blue-700 text-white rounded-xl font-medium transition-all duration-300 transform hover:scale-105 shadow-lg"
              @click="downloadResults"
            >
              <span class="flex items-center">
                <span class="text-xl mr-2">💾</span>
                下载结果
              </span>
            </button>
            <button 
              class="px-6 py-3 bg-gradient-to-r from-red-500 to-red-600 hover:from-red-600 hover:to-red-700 text-white rounded-xl font-medium transition-all duration-300 transform hover:scale-105 shadow-lg"
              @click="deleteRecord"
            >
              <span class="flex items-center">
                <span class="text-xl mr-2">🗑️</span>
                删除记录
              </span>
            </button>
          </div>
        </div>
      </div>
    </div>

    <!-- 图片查看模态框 -->
    <div v-if="modalImage" class="fixed inset-0 bg-black/80 flex items-center justify-center z-60 p-4">
      <div class="bg-base-100 rounded-2xl max-w-5xl max-h-[90vh] overflow-hidden">
        <div class="p-4">
          <div class="flex items-center justify-between mb-4">
            <h3 class="font-bold text-lg text-base-content">{{ modalTitle }}</h3>
            <button 
              class="w-8 h-8 bg-base-200 hover:bg-base-300 rounded-full flex items-center justify-center transition-colors"
              @click="closeImageModal"
            >
              ✕
            </button>
          </div>
          <img :src="modalImage" alt="查看大图" class="max-w-full max-h-[70vh] rounded-xl">
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, h, defineComponent, onMounted, onBeforeUnmount } from 'vue'
import { Motion } from 'motion-v'
import { segmentLiverTumor  } from '@/api'
import type { SegmentResponse } from '@/types/apis/pagesApi_T'
import { ElMessage } from 'element-plus'

// RevealMotion 组件定义
type RevealProps = { delay?: number }
const RevealMotion = defineComponent<RevealProps>({
  name: 'RevealMotion',
  props: { delay: { type: Number, default: 0 } },
  setup(props, { slots }) {
    const el = ref<HTMLElement | null>(null)
    const inView = ref(false)
    let io: IntersectionObserver | null = null

    const animateProps = computed(() => {
      return inView.value
        ? { opacity: 1, y: 0, transition: { duration: 0.6, delay: props.delay } }
        : { opacity: 0, y: 16 }
    })

    onMounted(() => {
      io = new IntersectionObserver(
        (entries) => {
          entries.forEach((e) => {
            if (e.isIntersecting) {
              inView.value = true
              io?.unobserve(e.target)
            }
          })
        },
        { threshold: 0.15 }
      )
      if (el.value) io.observe(el.value)
    })

    onBeforeUnmount(() => io?.disconnect())

    return () =>
      h(
        'div',
        { ref: el },
        [
          h(
            Motion as any,
            {
              initial: { opacity: 0, y: 16 },
              animate: animateProps.value,
            },
            slots
          )
        ]
      )
  },
})

// 响应式数据
const selectedFile = ref<File | null>(null)
const previewUrl = ref<string>('')
const isDragOver = ref(false)
const uploadError = ref<string>('')
const isLoading = ref(false)
const modalImage = ref<string>('')
const modalTitle = ref<string>('')
const fileInput = ref<HTMLInputElement>()
const historyRecords = ref<Array<SegmentResponse & { timestamp: number; original_filename?: string }>>([])
const showResultModal = ref(false)
const currentResult = ref<SegmentResponse & { timestamp: number; original_filename?: string }>({} as any)

// 文件选择处理
const handleFileSelect = (event: Event) => {
  const target = event.target as HTMLInputElement
  if (target.files && target.files[0]) {
    validateAndSetFile(target.files[0])
  }
}

// 拖拽处理
const handleDrop = (event: DragEvent) => {
  event.preventDefault()
  isDragOver.value = false
  
  if (event.dataTransfer?.files && event.dataTransfer.files[0]) {
    validateAndSetFile(event.dataTransfer.files[0])
  }
}

// 文件验证和设置
const validateAndSetFile = (file: File) => {
  uploadError.value = ''
  
  // 检查文件类型
  if (!file.type.startsWith('image/')) {
    uploadError.value = '请选择图片文件'
    return
  }
  
  // 检查文件大小 (20MB)
  if (file.size > 20 * 1024 * 1024) {
    uploadError.value = '文件大小不能超过 20MB'
    return
  }
  
  selectedFile.value = file
  
  // 创建预览URL
  const reader = new FileReader()
  reader.onload = (e) => {
    previewUrl.value = e.target?.result as string
  }
  reader.readAsDataURL(file)
}

// 清除文件
const clearFile = () => {
  selectedFile.value = null
  previewUrl.value = ''
  uploadError.value = ''
  if (previewUrl.value) {
    URL.revokeObjectURL(previewUrl.value)
  }
}

// 执行分割
const performSegmentation = async () => {
  if (!selectedFile.value) return
  
  isLoading.value = true
  uploadError.value = ''
  
  try {
    const formData = new FormData()
    formData.append('image', selectedFile.value)
    
    const response = await segmentLiverTumor (formData) 
    
    if (response.code === 200) {
      const resultWithTimestamp = {
        ...response.data,
        timestamp: Date.now(),
        original_filename: selectedFile.value.name
      }
      
      // 添加到历史记录
      historyRecords.value.unshift(resultWithTimestamp)
      
      // 限制历史记录数量
      if (historyRecords.value.length > 20) {
        historyRecords.value = historyRecords.value.slice(0, 20)
      }
      
      // 显示结果弹窗
      currentResult.value = resultWithTimestamp
      showResultModal.value = true
      
      // 清除当前选择的文件
      clearFile()
      
      ElMessage.success('🎉 AI检测完成！')
    } else {
      throw new Error(response.msg || '分割失败')
    }
  } catch (error: any) {
    console.error('分割错误:', error)
    uploadError.value = error.message || '分割过程中发生错误，请重试'
    ElMessage.error(uploadError.value)
  } finally {
    isLoading.value = false
  }
}

// 重置表单
const resetForm = () => {
  clearFile()
  uploadError.value = ''
}

// 打开结果弹窗
const openResultModal = (result: SegmentResponse & { timestamp: number; original_filename?: string }) => {
  currentResult.value = result
  showResultModal.value = true
}

// 关闭结果弹窗
const closeResultModal = () => {
  showResultModal.value = false
}

// 格式化日期
const formatDate = (timestamp: number): string => {
  const date = new Date(timestamp)
  const now = new Date()
  const diff = now.getTime() - date.getTime()
  
  if (diff < 60000) {
    return '刚刚'
  } else if (diff < 3600000) {
    return `${Math.floor(diff / 60000)}分钟前`
  } else if (diff < 86400000) {
    return `${Math.floor(diff / 3600000)}小时前`
  } else {
    return date.toLocaleDateString('zh-CN', {
      month: 'short',
      day: 'numeric',
      hour: '2-digit',
      minute: '2-digit'
    })
  }
}

// 清空历史记录
const clearHistory = () => {
  historyRecords.value = []
  ElMessage.success('历史记录已清空')
}

// 删除单条记录
const deleteRecord = () => {
  const index = historyRecords.value.findIndex(record => record.timestamp === currentResult.value.timestamp)
  if (index > -1) {
    historyRecords.value.splice(index, 1)
    closeResultModal()
    ElMessage.success('记录已删除')
  }
}

// 格式化文件大小
const formatFileSize = (bytes: number): string => {
  if (bytes === 0) return '0 Bytes'
  const k = 1024
  const sizes = ['Bytes', 'KB', 'MB', 'GB']
  const i = Math.floor(Math.log(bytes) / Math.log(k))
  return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + ' ' + sizes[i]
}

// 获取图片URL
const getImageUrl = (path: string): string => {
  const baseUrl = import.meta.env.VITE_SERVER_PATH || 'http://localhost:8000'
  return `${baseUrl}/${path}`
}

// 打开图片模态框
const openImageModal = (imageUrl: string, title: string) => {
  modalImage.value = imageUrl
  modalTitle.value = title
}

// 关闭图片模态框
const closeImageModal = () => {
  modalImage.value = ''
  modalTitle.value = ''
}

// 下载结果
const downloadResults = () => {
  const result = currentResult.value
  if (!result) return
  
  // 创建下载链接
  const downloadLink = (url: string, filename: string) => {
    const link = document.createElement('a')
    link.href = url
    link.download = filename
    document.body.appendChild(link)
    link.click()
    document.body.removeChild(link)
  }
  
  // 下载所有结果图片
  downloadLink(getImageUrl(result.original_image_path), 'original.jpg')
  downloadLink(getImageUrl(result.mask_image_path), 'mask.png')
  downloadLink(getImageUrl(result.overlay_image_path), 'overlay.jpg')
  
  ElMessage.success('💾 开始下载结果文件')
}
</script>

<style scoped>
/* 现代化样式 */
.bg-gradient-to-br {
  background-attachment: fixed;
}

/* 毛玻璃效果 */
.backdrop-blur-sm {
  backdrop-filter: blur(8px);
}

/* 动画效果 */
@keyframes float {
  0%, 100% { transform: translateY(0px); }
  50% { transform: translateY(-10px); }
}

.animate-float {
  animation: float 3s ease-in-out infinite;
}

/* 渐变文字 */
.bg-clip-text {
  -webkit-background-clip: text;
  background-clip: text;
}

/* 自定义滚动条 */
.overflow-y-auto::-webkit-scrollbar {
  width: 6px;
}

.overflow-y-auto::-webkit-scrollbar-track {
  background: hsl(var(--b2));
  border-radius: 3px;
}

.overflow-y-auto::-webkit-scrollbar-thumb {
  background: hsl(var(--bc) / 0.3);
  border-radius: 3px;
}

.overflow-y-auto::-webkit-scrollbar-thumb:hover {
  background: hsl(var(--bc) / 0.5);
}

/* 卡片悬停效果 */
.hover\:scale-105:hover {
  transform: scale(1.05);
}

/* 按钮动画 */
.transform {
  transition: transform 0.3s ease;
}

/* 图片加载动画 */
img {
  transition: all 0.3s ease;
}

/* 弹窗动画 */
.fixed {
  animation: fadeIn 0.3s ease-out;
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: scale(0.9);
  }
  to {
    opacity: 1;
    transform: scale(1);
  }
}

/* 响应式设计 */
@media (max-width: 768px) {
  .grid-cols-1.lg\:grid-cols-3 {
    grid-template-columns: 1fr;
  }
  
  .text-5xl {
    font-size: 2.5rem;
  }
}
</style>
