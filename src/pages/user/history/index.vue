<template>
  <div class="min-h-screen bg-base-100 p-4">
    <!-- 页面头部 -->
    <RevealMotion>
      <div class="bg-base-200/80 backdrop-blur-sm rounded-3xl shadow-xl border border-base-300 p-8 mb-8">
        <div class="text-center">
          <h1 class="text-4xl font-bold text-primary mb-4">
            🛣️ 道路缺陷检测历史
          </h1>
          <p class="text-xl text-base-content/70">智能分析记录 · 精准缺陷追踪</p>
          <div class="flex justify-center items-center mt-6 space-x-8">
            <div class="text-center">
              <div class="text-2xl font-bold text-primary">{{ historyData?.pagination?.total_count || 0 }}</div>
              <div class="text-sm text-base-content/60">总检测次数</div>
            </div>
            <div class="w-px h-8 bg-base-300"></div>
            <div class="text-center">
              <div class="text-2xl font-bold text-success">{{ getSuccessRate() }}%</div>
              <div class="text-sm text-base-content/60">检测成功率</div>
            </div>
            <div class="w-px h-8 bg-base-300"></div>
            <div class="text-center">
              <div class="text-2xl font-bold text-secondary">{{ getAverageConfidence() }}%</div>
              <div class="text-sm text-base-content/60">平均置信度</div>
            </div>
          </div>
        </div>
      </div>
    </RevealMotion>

    <!-- 搜索和筛选区域 -->
    <RevealMotion :delay="0.1">
      <div class="bg-base-200/80 backdrop-blur-sm rounded-2xl shadow-lg border border-base-300 p-6 mb-8">
        <div class="flex flex-col lg:flex-row gap-4">
          <!-- 搜索框 -->
          <div class="flex-1">
            <div class="relative">
              <input 
                v-model="searchQuery"
                type="text" 
                placeholder="搜索检测记录..."
                class="input input-bordered w-full pl-12 pr-4 py-3 bg-base-100 border border-base-300 rounded-xl focus:ring-2 focus:ring-primary focus:border-transparent transition-all"
                @input="handleSearch"
              />
              <div class="absolute left-4 top-1/2 transform -translate-y-1/2 text-base-content/50">
                🔍
              </div>
            </div>
          </div>
          
          <!-- 筛选器 -->
          <div class="flex flex-wrap gap-3">
            <select v-model="filters.page_size" class="select select-bordered px-4 py-3 bg-base-100 border border-base-300 rounded-xl focus:ring-2 focus:ring-primary focus:border-transparent">
              <option value="12">12条/页</option>
              <option value="24">24条/页</option>
              <option value="48">48条/页</option>
            </select>
            
            <input 
              v-model="filters.start_date" 
              type="date" 
              class="input input-bordered px-4 py-3 bg-base-100 border border-base-300 rounded-xl focus:ring-2 focus:ring-primary focus:border-transparent"
            />
            
            <input 
              v-model="filters.end_date" 
              type="date" 
              class="input input-bordered px-4 py-3 bg-base-100 border border-base-300 rounded-xl focus:ring-2 focus:ring-primary focus:border-transparent"
            />
            
            <button 
              @click="loadHistory" 
              :disabled="loading"
              class="btn btn-primary px-6 py-3 rounded-xl font-medium transition-all duration-300 transform hover:scale-105 shadow-lg disabled:opacity-50"
            >
              <span v-if="loading" class="animate-spin mr-2">⚡</span>
              {{ loading ? '搜索中...' : '🔍 搜索' }}
            </button>
            
            <button 
              @click="resetFilters" 
              class="btn btn-ghost px-6 py-3 rounded-xl font-medium transition-all duration-300"
            >
              🔄 重置
            </button>
          </div>
        </div>
      </div>
    </RevealMotion>

    <!-- 记录网格 -->
    <RevealMotion :delay="0.2">
      <!-- 加载状态 -->
      <div v-if="loading" class="flex justify-center items-center py-20">
        <div class="text-center">
          <div class="text-6xl mb-4 animate-bounce">🔄</div>
          <div class="text-xl font-medium text-base-content/70">正在加载检测记录...</div>
        </div>
      </div>

      <!-- 空状态 -->
      <div v-else-if="!historyData?.records?.length" class="text-center py-20">
        <div class="bg-base-200/80 backdrop-blur-sm rounded-3xl shadow-xl border border-base-300 p-12">
          <div class="text-8xl mb-6">🛣️</div>
          <h3 class="text-2xl font-bold text-base-content mb-4">暂无检测记录</h3>
          <p class="text-base-content/70 text-lg mb-8">还没有进行过道路缺陷检测，快去体验一下吧！</p>
          <button 
            @click="$router.push('/user/segmentation')"
            class="btn btn-primary px-8 py-4 rounded-xl font-medium transition-all duration-300 transform hover:scale-105 shadow-lg"
          >
            🚀 开始检测
          </button>
        </div>
      </div>

      <!-- 记录卡片网格 -->
      <div v-else class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-6">
        <div 
          v-for="(record, index) in historyData.records" 
          :key="record.id"
          class="card bg-base-200/80 backdrop-blur-sm rounded-2xl shadow-lg border border-base-300 overflow-hidden hover:shadow-2xl transition-all duration-300 transform hover:scale-105 cursor-pointer group"
          @click="showDetail(record)"
        >
          <!-- 卡片头部图片 -->
          <div class="relative h-48 overflow-hidden">
            <img 
              :src="getImageUrl(record.original_image)"
              :alt="`检测记录 ${record.id}`"
              class="w-full h-full object-cover group-hover:scale-110 transition-transform duration-300"
              @error="handleImageError"
              @load="handleImageLoad"
            />
            <div class="absolute top-3 right-3 bg-black/70 text-white text-xs px-3 py-1 rounded-full">
              #{{ record.id }}
            </div>
            <div class="absolute top-3 left-3 bg-gradient-to-r from-green-500 to-emerald-500 text-white text-xs px-3 py-1 rounded-full font-medium">
              {{ (record.confidence * 100).toFixed(1) }}% 置信度
            </div>
            <div class="absolute inset-0 bg-gradient-to-t from-black/50 to-transparent opacity-0 group-hover:opacity-100 transition-opacity duration-300"></div>
            <div class="absolute bottom-3 left-3 right-3 text-white opacity-0 group-hover:opacity-100 transition-opacity duration-300">
              <div class="text-sm font-medium">点击查看详情</div>
            </div>
          </div>
          
          <!-- 卡片内容 -->
          <div class="p-5">
            <div class="space-y-3">
              <!-- 缺陷比例进度条 -->
              <div>
                <div class="flex justify-between items-center mb-2">
                  <span class="text-sm font-medium text-gray-700">缺陷比例</span>
                  <span class="text-sm font-bold text-orange-600">{{ (record.tumor_ratio * 100).toFixed(2) }}%</span>
                </div>
                <div class="w-full bg-gray-200 rounded-full h-2">
                  <div 
                    class="bg-gradient-to-r from-orange-400 to-red-500 h-2 rounded-full transition-all duration-500"
                    :style="{ width: `${(record.tumor_ratio * 100).toFixed(1)}%` }"
                  ></div>
                </div>
              </div>
              
              <!-- 统计信息 -->
              <div class="grid grid-cols-2 gap-3">
                <div class="bg-primary/10 rounded-lg p-3 text-center">
                  <div class="text-xs text-base-content/70 mb-1">缺陷面积</div>
                  <div class="text-sm font-bold text-primary">{{ formatArea(record.tumor_area) }}</div>
                </div>
                <div class="bg-success/10 rounded-lg p-3 text-center">
                  <div class="text-xs text-base-content/70 mb-1">总面积</div>
                  <div class="text-sm font-bold text-success">{{ formatArea(record.total_area) }}</div>
                </div>
              </div>
              
              <!-- 时间信息 -->
              <div class="flex items-center justify-between pt-2 border-t border-base-300">
                <div class="flex items-center text-base-content/60 text-xs">
                  <span class="mr-1">🕒</span>
                  {{ formatDate(record.segmentation_time) }}
                </div>
                <div class="text-xs text-base-content/50">
                  {{ record.patient_id || '未指定ID' }}
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- 分页器 -->
      <div v-if="historyData?.pagination && historyData.pagination.total_pages > 1" class="mt-12">
        <div class="bg-base-200/80 backdrop-blur-sm rounded-2xl shadow-lg border border-base-300 p-6">
          <div class="flex flex-col sm:flex-row justify-between items-center gap-4">
            <div class="text-sm text-base-content/70">
              显示第 {{ (historyData.pagination.current_page - 1) * (filters.page_size || 12) + 1 }} - 
              {{ Math.min(historyData.pagination.current_page * (filters.page_size || 12), historyData.pagination.total_count) }} 条，
              共 {{ historyData.pagination.total_count }} 条记录
            </div>
            
            <div class="flex items-center space-x-2">
              <button 
                @click="changePage(historyData.pagination.current_page - 1)"
                :disabled="!historyData.pagination.has_previous"
                class="btn btn-outline btn-sm px-4 py-2 disabled:opacity-50 disabled:cursor-not-allowed rounded-lg transition-all"
              >
                ← 上一页
              </button>
              
              <div class="flex space-x-1">
                <button 
                  v-for="page in getPageNumbers()"
                  :key="page"
                  @click="changePage(page)"
                  :class="[
                    'btn btn-sm px-3 py-2 rounded-lg transition-all',
                    page === historyData.pagination.current_page 
                      ? 'btn-primary shadow-lg' 
                      : 'btn-outline'
                  ]"
                >
                  {{ page }}
                </button>
              </div>
              
              <button 
                @click="changePage(historyData.pagination.current_page + 1)"
                :disabled="!historyData.pagination.has_next"
                class="btn btn-outline btn-sm px-4 py-2 disabled:opacity-50 disabled:cursor-not-allowed rounded-lg transition-all"
              >
                下一页 →
              </button>
            </div>
          </div>
        </div>
      </div>
    </RevealMotion>

    <!-- 详情模态框 -->
    <dialog ref="detailModal" class="modal">
      <div class="modal-box max-w-6xl bg-base-100/95 backdrop-blur-sm border border-base-300">
        <form method="dialog">
          <button class="btn btn-sm btn-circle btn-ghost absolute right-4 top-4">✕</button>
        </form>
        
        <div class="mb-6">
          <h3 class="text-2xl font-bold text-primary mb-2">
            🛣️ 检测详情 - #{{ selectedRecord?.id }}
          </h3>
          <p class="text-base-content/70">道路缺陷检测结果详细信息</p>
        </div>
        
        <div v-if="selectedRecord" class="space-y-8">
          <!-- 统计卡片 -->
          <div class="grid grid-cols-2 md:grid-cols-4 gap-4">
            <div class="bg-success/10 rounded-2xl p-4 border border-success/20">
              <div class="flex items-center justify-between mb-2">
                <span class="text-success text-2xl">🎯</span>
                <span class="text-xs text-success bg-success/20 px-2 py-1 rounded-full">置信度</span>
              </div>
              <div class="text-2xl font-bold text-success">{{ (selectedRecord.confidence * 100).toFixed(1) }}%</div>
              <div class="text-xs text-success mt-1">检测准确性</div>
            </div>
            
            <div class="bg-warning/10 rounded-2xl p-4 border border-warning/20">
              <div class="flex items-center justify-between mb-2">
                <span class="text-warning text-2xl">⚠️</span>
                <span class="text-xs text-warning bg-warning/20 px-2 py-1 rounded-full">缺陷面积</span>
              </div>
              <div class="text-lg font-bold text-warning">{{ formatArea(selectedRecord.tumor_area) }}</div>
              <div class="text-xs text-warning mt-1">像素平方</div>
            </div>
            
            <div class="bg-info/10 rounded-2xl p-4 border border-info/20">
              <div class="flex items-center justify-between mb-2">
                <span class="text-info text-2xl">📐</span>
                <span class="text-xs text-info bg-info/20 px-2 py-1 rounded-full">总面积</span>
              </div>
              <div class="text-lg font-bold text-info">{{ formatArea(selectedRecord.total_area) }}</div>
              <div class="text-xs text-info mt-1">像素平方</div>
            </div>
            
            <div class="bg-secondary/10 rounded-2xl p-4 border border-secondary/20">
              <div class="flex items-center justify-between mb-2">
                <span class="text-secondary text-2xl">📊</span>
                <span class="text-xs text-secondary bg-secondary/20 px-2 py-1 rounded-full">缺陷比例</span>
              </div>
              <div class="text-lg font-bold text-secondary">{{ (selectedRecord.tumor_ratio * 100).toFixed(2) }}%</div>
              <div class="text-xs text-secondary mt-1">占总面积</div>
            </div>
          </div>

          <!-- 图片展示 -->
          <div class="space-y-6">
            <div class="flex items-center gap-3">
              <span class="text-2xl">🖼️</span>
              <h4 class="text-xl font-bold text-base-content">检测结果图像</h4>
            </div>
            
            <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
              <!-- 原始图像 -->
              <div class="bg-base-100/80 rounded-2xl p-4 shadow-lg border border-base-300">
                <h5 class="font-semibold text-base-content mb-3 flex items-center gap-2">
                  <span class="text-primary">📷</span>
                  原始道路图像
                </h5>
                <div class="relative group">
                  <div class="absolute inset-0 bg-base-200 rounded-xl flex items-center justify-center z-10">
                    <div class="animate-spin text-2xl">⚡</div>
                  </div>
                  <img 
                    :src="getImageUrl(selectedRecord.original_image)"
                    :data-original-path="selectedRecord.original_image"
                    alt="原始道路图像"
                    class="w-full h-56 object-cover rounded-xl border border-base-300 cursor-pointer hover:scale-105 transition-all duration-300 relative z-20 shadow-md"
                    style="opacity: 0; transition: opacity 0.3s ease;"
                    @click="showImageModal(getImageUrl(selectedRecord.original_image), '原始道路图像', $event)"
                    @error="handleImageError"
                    @load="handleImageLoad"
                    loading="lazy"
                  />
                  <div class="absolute inset-0 bg-black/0 group-hover:bg-black/20 transition-all rounded-xl flex items-center justify-center z-30 pointer-events-none">
                    <span class="text-white opacity-0 group-hover:opacity-100 transition-opacity font-medium">🔍 点击放大</span>
                  </div>
                </div>
              </div>

              <!-- 检测掩码 -->
              <div class="bg-white/80 rounded-2xl p-4 shadow-lg border border-white/20">
                <h5 class="font-semibold text-base-content mb-3 flex items-center gap-2">
                  <span class="text-success">🎭</span>
                  缺陷检测掩码
                </h5>
                <div class="relative group">
                  <div class="absolute inset-0 bg-gray-100 rounded-xl flex items-center justify-center z-10">
                    <div class="animate-spin text-2xl">⚡</div>
                  </div>
                  <img 
                    :src="getImageUrl(selectedRecord.mask_image)"
                    :data-original-path="selectedRecord.mask_image"
                    alt="缺陷检测掩码"
                    class="w-full h-56 object-cover rounded-xl border border-gray-200 cursor-pointer hover:scale-105 transition-all duration-300 relative z-20 shadow-md"
                    style="opacity: 0; transition: opacity 0.3s ease;"
                    @click="showImageModal(getImageUrl(selectedRecord.mask_image), '缺陷检测掩码', $event)"
                    @error="handleImageError"
                    @load="handleImageLoad"
                    loading="lazy"
                  />
                  <div class="absolute inset-0 bg-black/0 group-hover:bg-black/20 transition-all rounded-xl flex items-center justify-center z-30 pointer-events-none">
                    <span class="text-white opacity-0 group-hover:opacity-100 transition-opacity font-medium">🔍 点击放大</span>
                  </div>
                </div>
              </div>

              <!-- 叠加结果 -->
              <div class="bg-white/80 rounded-2xl p-4 shadow-lg border border-white/20">
                <h5 class="font-semibold text-base-content mb-3 flex items-center gap-2">
                  <span class="text-secondary">🔗</span>
                  叠加检测结果
                </h5>
                <div class="relative group">
                  <div class="absolute inset-0 bg-gray-100 rounded-xl flex items-center justify-center z-10">
                    <div class="animate-spin text-2xl">⚡</div>
                  </div>
                  <img 
                    :src="getImageUrl(selectedRecord.overlay_image)"
                    :data-original-path="selectedRecord.overlay_image"
                    alt="叠加检测结果"
                    class="w-full h-56 object-cover rounded-xl border border-gray-200 cursor-pointer hover:scale-105 transition-all duration-300 relative z-20 shadow-md"
                    style="opacity: 0; transition: opacity 0.3s ease;"
                    @click="showImageModal(getImageUrl(selectedRecord.overlay_image), '叠加检测结果', $event)"
                    @error="handleImageError"
                    @load="handleImageLoad"
                    loading="lazy"
                  />
                  <div class="absolute inset-0 bg-black/0 group-hover:bg-black/20 transition-all rounded-xl flex items-center justify-center z-30 pointer-events-none">
                    <span class="text-white opacity-0 group-hover:opacity-100 transition-opacity font-medium">🔍 点击放大</span>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <!-- 详细信息 -->
          <div class="bg-base-200/50 rounded-2xl p-6 border border-base-300">
            <div class="flex items-center gap-3 mb-4">
              <span class="text-2xl">📋</span>
              <h4 class="text-xl font-bold text-base-content">检测信息</h4>
            </div>
            
            <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
              <div class="space-y-3">
                <div class="flex items-center gap-3">
                  <span class="text-primary">🆔</span>
                  <span class="font-medium text-base-content">检测ID:</span>
                  <span class="text-base-content/70">{{ selectedRecord.patient_id || '未指定' }}</span>
                </div>
                <div class="flex items-center gap-3">
                  <span class="text-success">🔗</span>
                  <span class="font-medium text-base-content">会话ID:</span>
                  <code class="text-xs bg-base-200 px-2 py-1 rounded text-base-content/70">{{ selectedRecord.session_id }}</code>
                </div>
              </div>
              
              <div class="space-y-3">
                <div class="flex items-center gap-3">
                  <span class="text-secondary">🕒</span>
                  <span class="font-medium text-base-content">检测时间:</span>
                  <span class="text-base-content/70">{{ formatDate(selectedRecord.segmentation_time) }}</span>
                </div>
                <div v-if="selectedRecord.diagnosis_notes" class="flex items-start gap-3">
                  <span class="text-warning">📝</span>
                  <span class="font-medium text-base-content">备注:</span>
                  <span class="text-base-content/70">{{ selectedRecord.diagnosis_notes }}</span>
                </div>
              </div>
            </div>
          </div>

          <!-- 操作按钮 -->
          <div class="flex flex-wrap gap-3 pt-4 border-t border-base-300">
            <button 
              @click="downloadImage(getImageUrl(selectedRecord.original_image), '原始道路图像')" 
              class="btn btn-primary px-6 py-3 rounded-xl font-medium transition-all duration-300 transform hover:scale-105 shadow-lg flex items-center gap-2"
            >
              <span>📷</span>
              下载原始图像
            </button>
            <button 
              @click="downloadImage(getImageUrl(selectedRecord.mask_image), '缺陷检测掩码')" 
              class="btn btn-success px-6 py-3 rounded-xl font-medium transition-all duration-300 transform hover:scale-105 shadow-lg flex items-center gap-2"
            >
              <span>🎭</span>
              下载检测掩码
            </button>
            <button 
              @click="downloadImage(getImageUrl(selectedRecord.overlay_image), '叠加检测结果')" 
              class="btn btn-secondary px-6 py-3 rounded-xl font-medium transition-all duration-300 transform hover:scale-105 shadow-lg flex items-center gap-2"
            >
              <span>🔗</span>
              下载叠加结果
            </button>
          </div>
        </div>
      </div>
      <form method="dialog" class="modal-backdrop">
        <button>close</button>
      </form>
    </dialog>

    <!-- 图片放大模态框 -->
    <dialog ref="imageModal" class="modal">
      <div class="modal-box max-w-5xl bg-base-100/95 backdrop-blur-sm border border-base-300">
        <form method="dialog">
          <button class="btn btn-sm btn-circle btn-ghost absolute right-2 top-2 text-base-content/60 hover:text-base-content">✕</button>
        </form>
        
        <h3 class="font-bold text-lg mb-4 text-base-content">{{ currentImageTitle }}</h3>
        
        <div class="flex justify-center bg-base-200 rounded-xl p-4">
          <img 
            :src="currentImageUrl"
            :data-original-path="currentImageUrl"
            :alt="currentImageTitle"
            class="max-w-full max-h-[70vh] object-contain rounded-lg shadow-lg border border-base-300"
            style="opacity: 0; transition: opacity 0.3s ease;"
            @error="handleImageError"
            @load="handleImageLoad"
          />
        </div>
        
        <div class="flex justify-center mt-4 pt-4 border-t border-base-300">
          <button @click="downloadImage(currentImageUrl, currentImageTitle)" class="btn btn-primary px-6 py-3 rounded-xl font-medium transition-all duration-300 transform hover:scale-105 shadow-lg flex items-center gap-2">
            <span>📥</span>
            下载图片
          </button>
        </div>
      </div>
      <form method="dialog" class="modal-backdrop">
        <button>close</button>
      </form>
    </dialog>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, computed, h, defineComponent, onBeforeUnmount } from 'vue'
import { Motion } from 'motion-v'
import { ElMessage } from 'element-plus'
import { getSegmentationHistory } from '@/api'
import type { HistoryQueryParams, HistoryResponse, SegmentationRecord } from '@/types/apis/pagesApi_T'

// 动画组件
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
const loading = ref(false)
const historyData = ref<HistoryResponse | null>(null)
const selectedRecord = ref<SegmentationRecord | null>(null)
const detailModal = ref<HTMLDialogElement>()
const imageModal = ref<HTMLDialogElement>()
const currentImageUrl = ref('')
const currentImageTitle = ref('')

// 筛选器
const filters = ref<HistoryQueryParams>({
  page: 1,
  page_size: 20,
  patient_id: '',
  start_date: '',
  end_date: ''
})

// 服务器路径
const serverPath = import.meta.env.VITE_SERVER_PATH || 'http://localhost:8000'

// 加载历史记录
const loadHistory = async () => {
  try {
    loading.value = true
    
    // 清理空值
    const params: HistoryQueryParams = {
      page: filters.value.page,
      page_size: filters.value.page_size
    }
    
    if (filters.value.patient_id?.trim()) {
      params.patient_id = filters.value.patient_id.trim()
    }
    if (filters.value.start_date) {
      params.start_date = filters.value.start_date
    }
    if (filters.value.end_date) {
      params.end_date = filters.value.end_date
    }
    
    const response = await getSegmentationHistory(params)
    historyData.value = response.data
  } catch (error) {
    console.error('加载历史记录失败:', error)
    ElMessage.error('加载历史记录失败，请稍后重试')
  } finally {
    loading.value = false
  }
}

// 重置筛选器
const resetFilters = () => {
  filters.value = {
    page: 1,
    page_size: 20,
    patient_id: '',
    start_date: '',
    end_date: ''
  }
  loadHistory()
}

// 切换页面
const changePage = (page: number) => {
  if (page < 1 || (historyData.value?.pagination && page > historyData.value.pagination.total_pages)) {
    return
  }
  filters.value.page = page
  loadHistory()
}

// 获取页码数组
const getPageNumbers = () => {
  if (!historyData.value?.pagination) return []
  
  const { current_page, total_pages } = historyData.value.pagination
  const pages: number[] = []
  
  // 显示当前页前后各2页
  const start = Math.max(1, current_page - 2)
  const end = Math.min(total_pages, current_page + 2)
  
  for (let i = start; i <= end; i++) {
    pages.push(i)
  }
  
  return pages
}

// 显示详情
const showDetail = (record: SegmentationRecord) => {
  selectedRecord.value = record
  detailModal.value?.showModal()
}

// 显示图片模态框
const showImageModal = (url: string, title: string, event?: Event) => {
  // 检查图片是否加载失败
  if (event) {
    const img = event.target as HTMLImageElement
    if (img.dataset.loadFailed === 'true' || img.dataset.loadSuccess !== 'true') {
      console.warn('图片加载失败或未完成，无法放大显示')
      ElMessage.warning('图片加载失败，无法放大显示')
      return
    }
  }
  
  console.log('显示图片模态框:', url, title)
  currentImageUrl.value = url
  currentImageTitle.value = title
  
  // 确保模态框正确显示
  setTimeout(() => {
    imageModal.value?.showModal()
  }, 100)
}

// 获取图片URL
const getImageUrl = (imagePath: string) => {
  console.log('原始图片路径:', imagePath)
  console.log('服务器路径:', serverPath)
  
  if (!imagePath) return ''
  // 如果已经是完整URL，直接返回
  if (imagePath.startsWith('http')) return imagePath
  
  // 处理路径分隔符，统一使用正斜杠
  const normalizedPath = imagePath.replace(/\\/g, '/')
  // 移除开头的斜杠
  const cleanPath = normalizedPath.replace(/^\//, '')
  
  const finalUrl = `${serverPath}/${cleanPath}`
  console.log('最终图片URL:', finalUrl)
  
  return finalUrl
}

// 处理图片加载完成
const handleImageLoad = (event: Event) => {
  const img = event.target as HTMLImageElement
  console.log('图片加载成功:', img.src)
  
  // 标记图片加载成功
  img.dataset.loadSuccess = 'true'
  img.dataset.loadFailed = 'false'
  
  // 隐藏加载指示器
  const loadingIndicator = img.parentElement?.querySelector('.loading')
  if (loadingIndicator) {
    const loadingContainer = loadingIndicator.parentElement
    if (loadingContainer) {
      loadingContainer.style.display = 'none'
    }
  }
  
  // 确保图片可见
  img.style.opacity = '1'
  img.style.visibility = 'visible'
}

// 处理图片加载错误
const handleImageError = (event: Event) => {
  const img = event.target as HTMLImageElement
  const originalSrc = img.src
  
  console.error('图片加载失败:', originalSrc)
  
  // 如果还没有尝试过重试，则尝试重新构建URL
  if (!img.dataset.retried) {
    img.dataset.retried = 'true'
    
    // 尝试不同的路径格式
    const originalPath = img.getAttribute('data-original-path')
    if (originalPath) {
      // 尝试直接使用原始路径
      const retryUrl = `${serverPath}/${originalPath}`
      console.log('重试URL:', retryUrl)
      img.src = retryUrl
      return
    }
  }
  
  // 最终失败，标记为加载失败并显示占位图
  img.dataset.loadFailed = 'true'
  img.dataset.loadSuccess = 'false'
  
  // 隐藏加载指示器
  const loadingIndicator = img.parentElement?.querySelector('.loading')
  if (loadingIndicator) {
    const loadingContainer = loadingIndicator.parentElement
    if (loadingContainer) {
      loadingContainer.style.display = 'none'
    }
  }
  
  img.src = 'data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMjAwIiBoZWlnaHQ9IjIwMCIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj48cmVjdCB3aWR0aD0iMTAwJSIgaGVpZ2h0PSIxMDAlIiBmaWxsPSIjZjNmNGY2Ii8+PHRleHQgeD0iNTAlIiB5PSI1MCUiIGZvbnQtZmFtaWx5PSJBcmlhbCwgc2Fucy1zZXJpZiIgZm9udC1zaXplPSIxNCIgZmlsbD0iIzk5YTNhZiIgdGV4dC1hbmNob3I9Im1pZGRsZSIgZHk9Ii4zZW0iPuWbvueJh+WKoOi9veWksei0pTwvdGV4dD48L3N2Zz4='
  
  // 确保占位图可见
  img.style.opacity = '1'
  img.style.visibility = 'visible'
  
  // 移除点击事件和悬停效果
  img.style.cursor = 'default'
  img.onclick = null
  const parent = img.parentElement
  if (parent) {
    parent.classList.remove('cursor-pointer', 'hover:opacity-80')
    // 移除悬停文本
    const hoverOverlay = parent.querySelector('.absolute.inset-0')
    if (hoverOverlay) {
      hoverOverlay.remove()
    }
  }
  
  console.warn(`图片加载失败，已禁用点击放大: ${originalSrc}`)
}

// 下载图片
const downloadImage = async (url: string, filename: string) => {
  try {
    // 检查URL是否为占位图
    if (url.startsWith('data:image/svg+xml')) {
      ElMessage.warning('该图片加载失败，无法下载')
      return
    }
    
    const response = await fetch(url)
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`)
    }
    
    const blob = await response.blob()
    const downloadUrl = window.URL.createObjectURL(blob)
    
    const link = document.createElement('a')
    link.href = downloadUrl
    link.download = `${filename}_${Date.now()}.jpg`
    document.body.appendChild(link)
    link.click()
    document.body.removeChild(link)
    
    window.URL.revokeObjectURL(downloadUrl)
    ElMessage.success('图片下载成功')
  } catch (error) {
    console.error('下载图片失败:', error)
    ElMessage.error('下载图片失败，请稍后重试')
  }
}

// 格式化面积数值
const formatArea = (area: number) => {
  if (!area) return '0'
  return area.toLocaleString()
}

// 计算成功率
const getSuccessRate = () => {
  if (!historyData.value?.records?.length) return 0
  return 100 // 假设所有检测都成功
}

// 计算平均置信度
const getAverageConfidence = () => {
  if (!historyData.value?.records?.length) return 0
  const total = historyData.value.records.reduce((sum, record) => sum + (record.confidence * 100), 0)
  return Math.round(total / historyData.value.records.length)
}

// 搜索处理
const searchQuery = ref('')
const handleSearch = () => {
  filters.value.patient_id = searchQuery.value
  filters.value.page = 1
  loadHistory()
}

// 格式化日期
const formatDate = (dateString: string) => {
  const date = new Date(dateString)
  return date.toLocaleString('zh-CN', {
    year: 'numeric',
    month: '2-digit',
    day: '2-digit',
    hour: '2-digit',
    minute: '2-digit',
    second: '2-digit'
  })
}

// 组件挂载时加载数据
onMounted(() => {
  loadHistory()
})
</script>

<style scoped>
/* 自定义样式 */
.table th {
  @apply bg-base-300 text-base-content font-semibold;
}

.table td {
  @apply border-b border-base-200;
}

.stat {
  @apply p-4;
}

.stat-title {
  @apply text-xs text-base-content/70 font-medium;
}

.stat-value {
  @apply text-lg font-bold;
}

.stat-desc {
  @apply text-xs text-base-content/70;
}

/* 自定义滚动条 */
::-webkit-scrollbar {
  width: 8px;
  height: 8px;
}

::-webkit-scrollbar-track {
  background: hsl(var(--b2));
  border-radius: 4px;
}

::-webkit-scrollbar-thumb {
  background: hsl(var(--p));
  border-radius: 4px;
}

::-webkit-scrollbar-thumb:hover {
  background: hsl(var(--pf));
}
</style>
