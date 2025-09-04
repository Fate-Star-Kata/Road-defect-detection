<template>
  <div class="min-h-screen bg-base-100 text-base-content overflow-hidden">
    <!-- 背景装饰 -->
    
    <!-- 主容器 -->
    <div class="relative z-10">


      <!-- 仪表盘主体 -->
      <div class="container mx-auto px-6 py-8">
        <!-- 核心指标圆环 -->
        <RevealMotion :delay="0.1">
          <div class="text-center mb-12">
            <h2 class="text-3xl font-bold mb-8 text-base-content">
              模型性能概览
            </h2>
            
            <div v-if="performanceLoading || modelSettingsLoading" class="flex justify-center py-16">
              <div class="relative">
                <div class="loading loading-spinner loading-lg text-primary"></div>
                <div class="absolute inset-0 flex items-center justify-center mt-16">
                  <span class="text-sm font-medium text-base-content/70">加载中...</span>
                </div>
              </div>
            </div>
            
            <div v-else-if="performanceStats && modelSettings" class="grid grid-cols-1 md:grid-cols-3 gap-8 max-w-4xl mx-auto">
              <!-- 成功率圆环 -->
              <div class="relative">
                <div class="w-40 h-40 mx-auto relative">
                  <svg class="w-full h-full transform -rotate-90" viewBox="0 0 100 100">
                    <circle cx="50" cy="50" r="40" stroke="currentColor" stroke-width="8" fill="none" class="text-base-300"/>
                    <circle cx="50" cy="50" r="40" stroke="currentColor" stroke-width="8" fill="none" 
                            class="text-success" stroke-linecap="round"
                            :stroke-dasharray="`${performanceStats.overall_stats.overall_success_rate * 2.51} 251`"/>
                  </svg>
                  <div class="absolute inset-0 flex flex-col items-center justify-center">
                    <span class="text-3xl font-bold text-success">{{ performanceStats.overall_stats.overall_success_rate.toFixed(1) }}%</span>
                    <span class="text-sm text-base-content/70">成功率</span>
                  </div>
                </div>
              </div>
              
              <!-- 置信度圆环 -->
              <div class="relative">
                <div class="w-40 h-40 mx-auto relative">
                  <svg class="w-full h-full transform -rotate-90" viewBox="0 0 100 100">
                    <circle cx="50" cy="50" r="40" stroke="currentColor" stroke-width="8" fill="none" class="text-base-300"/>
                    <circle cx="50" cy="50" r="40" stroke="currentColor" stroke-width="8" fill="none" 
                            class="text-info" stroke-linecap="round"
                            :stroke-dasharray="`${performanceStats.overall_stats.overall_avg_confidence * 251} 251`"/>
                  </svg>
                  <div class="absolute inset-0 flex flex-col items-center justify-center">
                    <span class="text-3xl font-bold text-info">{{ (performanceStats.overall_stats.overall_avg_confidence * 100).toFixed(1) }}%</span>
                    <span class="text-sm text-base-content/70">平均置信度</span>
                  </div>
                </div>
              </div>
              
              <!-- 缺陷比例圆环 -->
              <div class="relative">
                <div class="w-40 h-40 mx-auto relative">
                  <svg class="w-full h-full transform -rotate-90" viewBox="0 0 100 100">
                    <circle cx="50" cy="50" r="40" stroke="currentColor" stroke-width="8" fill="none" class="text-base-300"/>
                    <circle cx="50" cy="50" r="40" stroke="currentColor" stroke-width="8" fill="none" 
                            class="text-warning" stroke-linecap="round"
                            :stroke-dasharray="`${performanceStats.overall_stats.overall_avg_tumor_ratio * 251} 251`"/>
                  </svg>
                  <div class="absolute inset-0 flex flex-col items-center justify-center">
                    <span class="text-3xl font-bold text-warning">{{ (performanceStats.overall_stats.overall_avg_tumor_ratio * 100).toFixed(1) }}%</span>
                    <span class="text-sm text-base-content/70">平均缺陷比例</span>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </RevealMotion>

        <!-- 模型配置时间线 -->
        <RevealMotion :delay="0.2">
          <div class="mb-12">
            <h3 class="text-2xl font-bold mb-8 text-center text-base-content">模型配置信息</h3>
            
            <div v-if="modelSettingsLoading" class="flex justify-center py-8">
              <div class="loading loading-spinner loading-md text-primary"></div>
            </div>
            
            <div v-else-if="modelSettings" class="max-w-4xl mx-auto">
              <div class="relative">
                <!-- 时间线 -->
                <div class="absolute left-1/2 transform -translate-x-1/2 h-full w-1 bg-base-300 rounded-full"></div>
                
                <!-- 配置项 -->
                <div class="space-y-12">
                  <!-- 置信度阈值 -->
                  <div class="flex items-center">
                    <div class="w-1/2 pr-8 text-right">
                      <div class="card bg-base-100 shadow-md border border-base-300">
                        <div class="card-body">
                          <h4 class="card-title text-base-content">置信度阈值</h4>
                          <div class="text-3xl font-bold text-info">{{ modelSettings.confidence_threshold }}</div>
                          <p class="text-sm text-base-content/70 mt-2">模型预测的最低置信度要求</p>
                        </div>
                      </div>
                    </div>
                    <div class="w-8 h-8 bg-info rounded-full border-4 border-base-100 flex items-center justify-center relative z-10 shadow-sm">
                      <div class="w-3 h-3 bg-base-100 rounded-full"></div>
                    </div>
                    <div class="w-1/2 pl-8"></div>
                  </div>
                  
                  <!-- 输入尺寸 -->
                  <div class="flex items-center">
                    <div class="w-1/2 pr-8"></div>
                    <div class="w-8 h-8 bg-secondary rounded-full border-4 border-base-100 flex items-center justify-center relative z-10 shadow-sm">
                      <div class="w-3 h-3 bg-base-100 rounded-full"></div>
                    </div>
                    <div class="w-1/2 pl-8">
                      <div class="card bg-base-100 shadow-md border border-base-300">
                        <div class="card-body">
                          <h4 class="card-title text-base-content">输入尺寸</h4>
                          <div class="text-3xl font-bold text-secondary">{{ modelSettings.input_size }}</div>
                          <p class="text-sm text-base-content/70 mt-2">图像输入的标准尺寸</p>
                        </div>
                      </div>
                    </div>
                  </div>
                  
                  <!-- 模型状态 -->
                  <div class="flex items-center">
                    <div class="w-1/2 pr-8 text-right">
                      <div class="card bg-base-100 shadow-md border border-base-300">
                        <div class="card-body">
                          <h4 class="card-title text-base-content">模型状态</h4>
                          <div class="flex items-center justify-end space-x-2">
                            <div class="w-3 h-3 rounded-full" :class="modelSettings.is_active ? 'bg-success animate-pulse' : 'bg-error'"></div>
                            <span class="text-xl font-bold" :class="modelSettings.is_active ? 'text-success' : 'text-error'">
                              {{ modelSettings.is_active ? '激活' : '未激活' }}
                            </span>
                          </div>
                          <p class="text-sm text-base-content/70 mt-2">当前模型运行状态</p>
                        </div>
                      </div>
                    </div>
                    <div class="w-8 h-8 rounded-full border-4 border-base-100 flex items-center justify-center relative z-10 shadow-sm"
                         :class="modelSettings.is_active ? 'bg-success' : 'bg-error'">
                      <div class="w-3 h-3 bg-base-100 rounded-full"></div>
                    </div>
                    <div class="w-1/2 pl-8"></div>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </RevealMotion>

        <!-- 性能趋势图表区域 -->
        <RevealMotion :delay="0.3">
          <div class="mb-12">
            <div class="flex items-center justify-between mb-8">
              <h3 class="text-2xl font-bold text-base-content">性能趋势分析</h3>
              <select class="select select-bordered bg-base-100 border-base-300 text-base-content" v-model="selectedDays" @change="loadPerformanceStats">
                <option value="7">最近7天</option>
                <option value="30">最近30天</option>
                <option value="90">最近90天</option>
              </select>
            </div>
            
            <div v-if="performanceLoading" class="flex justify-center py-16">
              <div class="loading loading-spinner loading-md text-primary"></div>
            </div>
            
            <div v-else-if="performanceStats" class="card bg-base-100 shadow-lg border border-base-300">
              <div class="card-body">
              <!-- 统计卡片网格 -->
              <div class="grid grid-cols-1 md:grid-cols-4 gap-6 mb-8">
                <div class="stats shadow border border-base-300">
                  <div class="stat">
                    <div class="stat-title text-base-content/70">总检测次数</div>
                    <div class="stat-value text-primary">{{ performanceStats.overall_stats.total_segmentations }}</div>
                  </div>
                </div>
                <div class="stats shadow border border-base-300">
                  <div class="stat">
                    <div class="stat-title text-base-content/70">成功率</div>
                    <div class="stat-value text-success">{{ performanceStats.overall_stats.overall_success_rate.toFixed(1) }}%</div>
                  </div>
                </div>
                <div class="stats shadow border border-base-300">
                  <div class="stat">
                    <div class="stat-title text-base-content/70">平均置信度</div>
                    <div class="stat-value text-info">{{ performanceStats.overall_stats.overall_avg_confidence.toFixed(3) }}</div>
                  </div>
                </div>
                <div class="stats shadow border border-base-300">
                  <div class="stat">
                    <div class="stat-title text-base-content/70">平均缺陷比例</div>
                    <div class="stat-value text-warning">{{ (performanceStats.overall_stats.overall_avg_tumor_ratio * 100).toFixed(1) }}%</div>
                  </div>
                </div>
              </div>
              
              <!-- 每日数据列表 -->
              <div class="space-y-3">
                <h4 class="text-lg font-semibold mb-4 text-base-content">每日性能详情</h4>
                <div class="max-h-64 overflow-y-auto space-y-2">
                  <div v-for="stat in performanceStats.daily_stats" :key="stat.date" 
                       class="flex items-center justify-between p-4 bg-base-200 rounded-xl border border-base-300 hover:bg-base-300 transition-colors">
                    <div class="flex items-center space-x-4">
                      <div class="avatar placeholder">
                        <div class="bg-neutral text-neutral-content rounded-xl w-12">
                          <span class="text-sm font-bold">{{ formatDate(stat.date).split('/')[2] }}</span>
                        </div>
                      </div>
                      <div>
                        <div class="font-medium text-base-content">{{ formatDate(stat.date) }}</div>
                        <div class="text-sm text-base-content/70">{{ stat.total_segmentations }} 次检测</div>
                      </div>
                    </div>
                    <div class="flex items-center space-x-6">
                      <div class="text-center">
                        <div class="text-lg font-bold text-success">{{ stat.success_rate.toFixed(1) }}%</div>
                        <div class="text-xs text-base-content/70">成功率</div>
                      </div>
                      <div class="text-center">
                        <div class="text-lg font-bold text-info">{{ stat.avg_confidence.toFixed(3) }}</div>
                        <div class="text-xs text-base-content/70">置信度</div>
                      </div>
                    </div>
                  </div>
                </div>
              </div>
              
              <!-- 时间范围信息 -->
                <div class="text-center mt-6 pt-6 border-t border-base-300">
                  <p class="text-sm text-base-content/70">
                    统计时间范围：{{ formatDate(performanceStats.date_range.start_date) }} 至 {{ formatDate(performanceStats.date_range.end_date) }}
                    （{{ performanceStats.date_range.days }} 天）
                  </p>
                </div>
              </div>
            </div>
          </div>
        </RevealMotion>

        <!-- 模型时间信息 -->
        <RevealMotion :delay="0.4">
          <div v-if="modelSettings" class="bg-white/5 backdrop-blur-md rounded-3xl p-8 border border-white/10">
            <h3 class="text-xl font-bold mb-6 text-center">模型时间信息</h3>
            <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
              <div class="text-center p-6 bg-gradient-to-br from-indigo-500/20 to-indigo-600/20 rounded-2xl border border-indigo-400/30">
                <div class="text-lg font-semibold text-indigo-400 mb-2">创建时间</div>
                <div class="text-sm text-white/70">{{ formatDateTime(modelSettings.created_time) }}</div>
              </div>
              <div class="text-center p-6 bg-gradient-to-br from-teal-500/20 to-teal-600/20 rounded-2xl border border-teal-400/30">
                <div class="text-lg font-semibold text-teal-400 mb-2">更新时间</div>
                <div class="text-sm text-white/70">{{ formatDateTime(modelSettings.updated_time) }}</div>
              </div>
            </div>
          </div>
        </RevealMotion>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, h, defineComponent, computed } from 'vue'
import { Motion } from 'motion-v'
import { ElMessage } from 'element-plus'
import { getModelSettings, getModelPerformance } from '@/api'
import type { ModelSettings, PerformanceResponse } from '@/types/apis/pagesApi_T'

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
const modelSettings = ref<ModelSettings | null>(null)
const modelSettingsLoading = ref(false)
const performanceStats = ref<PerformanceResponse | null>(null)
const performanceLoading = ref(false)
const selectedDays = ref(7)

// 时间范围选项
const timeRangeOptions = [
  { value: 7, label: '最近7天' },
  { value: 30, label: '最近30天' },
  { value: 90, label: '最近90天' }
]

// 计算圆环进度条的路径
const getCirclePath = (percentage: number) => {
  const radius = 40
  const circumference = 2 * Math.PI * radius
  const strokeDasharray = circumference
  const strokeDashoffset = circumference - (percentage / 100) * circumference
  return { strokeDasharray, strokeDashoffset }
}

// 计算成功率
const successRate = computed(() => {
  if (!performanceStats.value?.overall_stats?.total_segmentations) return 0
  return performanceStats.value.overall_stats.overall_success_rate
})

// 计算平均置信度
const avgConfidence = computed(() => {
  if (!performanceStats.value?.overall_stats) return 0
  return performanceStats.value.overall_stats.overall_avg_confidence
})

// 计算平均缺陷比例
const avgDefectRatio = computed(() => {
  if (!performanceStats.value?.overall_stats) return 0
  return performanceStats.value.overall_stats.overall_avg_tumor_ratio
})

// 模型状态颜色
const getStatusColor = (isActive: boolean) => {
  return isActive ? 'text-green-400' : 'text-red-400'
}

// 模型状态文本
const getStatusText = (isActive: boolean) => {
  return isActive ? '运行中' : '已停用'
}

// 格式化数字
const formatNumber = (value: number | null | undefined) => {
  if (value === null || value === undefined) return '0'
  return value.toLocaleString()
}

// 加载模型设置
const loadModelSettings = async () => {
  try {
    modelSettingsLoading.value = true
    const response = await getModelSettings()
    if (response.code === 200) {
      modelSettings.value = response.data
    } else {
      throw new Error(response.msg || '获取模型设置失败')
    }
  } catch (error) {
    console.error('加载模型设置失败:', error)
    ElMessage.error('加载模型设置失败')
  } finally {
    modelSettingsLoading.value = false
  }
}

// 加载性能统计
const loadPerformanceStats = async () => {
  try {
    performanceLoading.value = true
    const response = await getModelPerformance({ days: selectedDays.value })
    if (response.code === 200) {
      performanceStats.value = response.data
    } else {
      throw new Error(response.msg || '获取性能统计失败')
    }
  } catch (error) {
    console.error('加载性能统计失败:', error)
    ElMessage.error('加载性能统计失败')
  } finally {
    performanceLoading.value = false
  }
}

// 格式化日期时间
const formatDateTime = (dateString: string) => {
  return new Date(dateString).toLocaleString('zh-CN', {
    year: 'numeric',
    month: '2-digit',
    day: '2-digit',
    hour: '2-digit',
    minute: '2-digit',
    second: '2-digit'
  })
}

// 格式化日期
const formatDate = (dateString: string) => {
  return new Date(dateString).toLocaleDateString('zh-CN', {
    year: 'numeric',
    month: '2-digit',
    day: '2-digit'
  })
}

// 时间范围改变处理
const handleTimeRangeChange = () => {
  loadPerformanceStats()
}

// 组件挂载时加载数据
onMounted(() => {
  loadModelSettings()
  loadPerformanceStats()
})
</script>

<style scoped>
/* 自定义样式 */
.loading {
  @apply inline-block animate-spin rounded-full border-4 border-solid border-current border-r-transparent align-[-0.125em] motion-reduce:animate-[spin_1.5s_linear_infinite];
}

.loading-spinner {
  @apply border-2;
}

.loading-lg {
  @apply h-8 w-8;
}

.select {
  @apply inline-flex h-12 min-h-12 flex-shrink-0 cursor-pointer select-none flex-wrap items-center justify-between rounded-btn border border-neutral bg-base-100 px-4 text-sm leading-loose transition duration-200 ease-in-out;
}

.select-bordered {
  @apply border-base-content/20;
}

/* 圆环进度条动画 */
@keyframes dash {
  to {
    stroke-dashoffset: 0;
  }
}

.progress-circle {
  animation: dash 2s ease-in-out;
}

/* 玻璃拟态效果 */
.glass {
  backdrop-filter: blur(16px) saturate(180%);
  background-color: rgba(255, 255, 255, 0.1);
  border: 1px solid rgba(255, 255, 255, 0.2);
}

/* 悬停效果 */
.hover-glow:hover {
  box-shadow: 0 0 20px rgba(59, 130, 246, 0.3);
  transform: translateY(-2px);
  transition: all 0.3s ease;
}

/* 渐变文字 */
.gradient-text {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

/* 脉冲动画 */
@keyframes pulse {
  0%, 100% {
    opacity: 1;
  }
  50% {
    opacity: 0.5;
  }
}

.animate-pulse {
  animation: pulse 2s cubic-bezier(0.4, 0, 0.6, 1) infinite;
}

/* 自定义滚动条 - 适配主题 */
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
</style>
