<script setup lang="ts">
import { ref, computed, onMounted, nextTick, watch } from 'vue'
import { ElMessage } from 'element-plus'
import { Refresh, DataAnalysis, User, TrendCharts, PieChart } from '@element-plus/icons-vue'
import { use } from 'echarts/core'
import { CanvasRenderer } from 'echarts/renderers'
import { LineChart, BarChart, PieChart as EChartsPieChart } from 'echarts/charts'
import {
  TitleComponent,
  TooltipComponent,
  LegendComponent,
  GridComponent
} from 'echarts/components'
import VChart from 'vue-echarts'

// 注册 ECharts 组件
use([
  CanvasRenderer,
  LineChart,
  BarChart,
  EChartsPieChart,
  TitleComponent,
  TooltipComponent,
  LegendComponent,
  GridComponent
])

// 接口定义
interface DailyStats {
  date: string
  segmentation_count: number
  avg_confidence: number
}

interface PatientStats {
  patient_id: string
  segmentation_count: number
  avg_confidence: number
}

interface DateRange {
  start_date: string
  end_date: string
  days: number
}

interface StatsData {
  total_segmentations: number
  daily_stats: DailyStats[]
  patient_stats: PatientStats[]
  date_range: DateRange
  confidence_distribution: { range: string; count: number }[]
}

// 响应式数据
const loading = ref(false)
const error = ref('')
const statsData = ref<StatsData | null>(null)
const activeTab = ref('trend')

// 统计卡片数据
const statsCards = computed(() => {
  if (!statsData.value) return []
  
  return [
    {
      key: 'total',
      label: '总分割次数',
      value: statsData.value.total_segmentations.toLocaleString(),
      icon: 'DataAnalysis',
      colorClass: 'stat-blue',
      trend: '本月新增 +12%'
    },
    {
      key: 'patients',
      label: '患者数量',
      value: statsData.value.patient_stats.length.toString(),
      icon: 'User',
      colorClass: 'stat-green',
      trend: '活跃用户占比 85%'
    },
    {
      key: 'days',
      label: '统计天数',
      value: statsData.value.date_range.days.toString(),
      icon: 'TrendCharts',
      colorClass: 'stat-orange',
      trend: '数据覆盖完整'
    },
    {
      key: 'active',
      label: '活跃天数',
      value: statsData.value.daily_stats.length.toString(),
      icon: 'PieChart',
      colorClass: 'stat-purple',
      trend: '使用率 ' + Math.round((statsData.value.daily_stats.length / statsData.value.date_range.days) * 100) + '%'
    }
  ]
})

// 图表配置
const dailyStatsOption = ref({})
const confidenceDistributionOption = ref({})
const patientStatsOption = ref({})

// 监听activeTab变化，重新渲染图表
watch(activeTab, async () => {
  if (statsData.value) {
    await nextTick()
    updateCharts()
  }
})

// 页面初始化
onMounted(() => {
  fetchStats()
})

// 生成模拟数据
const generateMockData = (): StatsData => {
  const days = 30
  const startDate = new Date()
  startDate.setDate(startDate.getDate() - days)
  
  const dailyStats: DailyStats[] = []
  const patientStats: PatientStats[] = []
  
  // 生成每日统计数据
  for (let i = 0; i < days; i++) {
    const date = new Date(startDate)
    date.setDate(date.getDate() + i)
    
    dailyStats.push({
      date: date.toISOString().split('T')[0],
      segmentation_count: Math.floor(Math.random() * 50) + 10,
      avg_confidence: Math.random() * 0.3 + 0.7
    })
  }
  
  // 生成患者统计数据
  for (let i = 1; i <= 15; i++) {
    patientStats.push({
      patient_id: `Patient_${i.toString().padStart(3, '0')}`,
      segmentation_count: Math.floor(Math.random() * 20) + 5,
      avg_confidence: Math.random() * 0.2 + 0.75
    })
  }
  
  // 生成置信度分布数据
  const confidenceDistribution = [
    { range: '0.9-1.0', count: Math.floor(Math.random() * 200) + 300 },
    { range: '0.8-0.9', count: Math.floor(Math.random() * 150) + 200 },
    { range: '0.7-0.8', count: Math.floor(Math.random() * 100) + 100 },
    { range: '0.6-0.7', count: Math.floor(Math.random() * 50) + 30 },
    { range: '0.5-0.6', count: Math.floor(Math.random() * 30) + 10 }
  ]
  
  const totalSegmentations = dailyStats.reduce((sum, day) => sum + day.segmentation_count, 0)
  
  return {
    total_segmentations: totalSegmentations,
    daily_stats: dailyStats,
    patient_stats: patientStats,
    date_range: {
      start_date: startDate.toISOString().split('T')[0],
      end_date: new Date().toISOString().split('T')[0],
      days: days
    },
    confidence_distribution: confidenceDistribution
  }
}

// 获取统计数据
const fetchStats = async () => {
  try {
    loading.value = true
    error.value = ''
    
    // 模拟API调用延迟
    await new Promise(resolve => setTimeout(resolve, 1000))
    
    // 使用模拟数据
    statsData.value = generateMockData()
    
    // 等待DOM更新后再更新图表
    await nextTick()
    updateCharts()
    
    ElMessage.success('数据刷新成功')
  } catch (err: any) {
    error.value = err.message || '网络请求失败'
    console.error('获取统计数据失败:', err)
    ElMessage.error('数据获取失败')
  } finally {
    loading.value = false
  }
}

// 更新图表配置
const updateCharts = () => {
  if (!statsData.value) return

  // 每日统计趋势图
  dailyStatsOption.value = {
    title: {
      text: '每日分割统计趋势',
      left: 'center',
      textStyle: {
        fontSize: 16,
        fontWeight: 'bold'
      }
    },
    tooltip: {
      trigger: 'axis',
      axisPointer: {
        type: 'cross'
      },
      backgroundColor: 'rgba(255, 255, 255, 0.95)',
      borderColor: '#e4e7ed',
      textStyle: {
        color: '#606266'
      }
    },
    legend: {
      data: ['分割次数', '平均置信度'],
      top: 30,
      textStyle: {
        color: '#606266'
      }
    },
    grid: {
      left: '3%',
      right: '4%',
      bottom: '3%',
      containLabel: true
    },
    xAxis: {
      type: 'category',
      data: statsData.value.daily_stats.map(item => {
        const date = new Date(item.date)
        return `${date.getMonth() + 1}/${date.getDate()}`
      }),
      axisLine: {
        lineStyle: {
          color: '#e4e7ed'
        }
      },
      axisLabel: {
        color: '#909399'
      }
    },
    yAxis: [
      {
        type: 'value',
        name: '分割次数',
        position: 'left',
        axisLine: {
          lineStyle: {
            color: '#409EFF'
          }
        },
        axisLabel: {
          color: '#909399'
        },
        splitLine: {
          lineStyle: {
            color: '#f5f7fa'
          }
        }
      },
      {
        type: 'value',
        name: '平均置信度',
        position: 'right',
        min: 0.5,
        max: 1,
        axisLine: {
          lineStyle: {
            color: '#67C23A'
          }
        },
        axisLabel: {
          color: '#909399',
          formatter: '{value}'
        }
      }
    ],
    series: [
      {
        name: '分割次数',
        type: 'bar',
        data: statsData.value.daily_stats.map(item => item.segmentation_count),
        itemStyle: {
          color: '#409EFF'
        },
        barWidth: '60%'
      },
      {
        name: '平均置信度',
        type: 'line',
        yAxisIndex: 1,
        data: statsData.value.daily_stats.map(item => Number(item.avg_confidence.toFixed(3))),
        itemStyle: {
          color: '#67C23A'
        },
        lineStyle: {
          width: 3
        },
        symbol: 'circle',
        symbolSize: 6
      }
    ]
  }

  // 置信度分布饼图
  confidenceDistributionOption.value = {
    title: {
      text: '置信度分布',
      left: 'center',
      top: '5%',
      textStyle: {
        fontSize: 16,
        fontWeight: 'bold'
      }
    },
    tooltip: {
      trigger: 'item',
      formatter: '{b}: {c} ({d}%)',
      backgroundColor: 'rgba(255, 255, 255, 0.95)',
      borderColor: '#e4e7ed',
      textStyle: {
        color: '#606266'
      }
    },
    legend: {
      orient: 'horizontal',
      left: 'center',
      bottom: '5%',
      textStyle: {
        color: '#606266'
      }
    },
    series: [
      {
        name: '置信度分布',
        type: 'pie',
        radius: ['30%', '60%'],
        center: ['50%', '55%'],
        data: statsData.value.confidence_distribution.map((item, index) => ({
          value: item.count,
          name: item.range,
          itemStyle: {
            color: ['#409EFF', '#67C23A', '#E6A23C', '#F56C6C', '#909399'][index]
          }
        })),
        emphasis: {
          itemStyle: {
            shadowBlur: 10,
            shadowOffsetX: 0,
            shadowColor: 'rgba(0, 0, 0, 0.5)'
          }
        },
        label: {
          show: true,
          formatter: '{b}\n{d}%',
          fontSize: 12
        }
      }
    ]
  }

  // 患者统计柱状图
  patientStatsOption.value = {
    title: {
      text: '患者分割统计',
      left: 'center',
      top: '5%',
      textStyle: {
        fontSize: 16,
        fontWeight: 'bold'
      }
    },
    tooltip: {
      trigger: 'axis',
      axisPointer: {
        type: 'shadow'
      },
      backgroundColor: 'rgba(255, 255, 255, 0.95)',
      borderColor: '#e4e7ed',
      textStyle: {
        color: '#606266'
      }
    },
    legend: {
      data: ['分割次数', '平均置信度'],
      top: '12%',
      textStyle: {
        color: '#606266'
      }
    },
    grid: {
      left: '8%',
      right: '8%',
      top: '20%',
      bottom: '15%',
      containLabel: true
    },
    xAxis: {
      type: 'category',
      data: statsData.value.patient_stats.slice(0, 10).map(item => item.patient_id || '未知患者'),
      axisLine: {
        lineStyle: {
          color: '#e4e7ed'
        }
      },
      axisLabel: {
        color: '#909399',
        rotate: 45,
        fontSize: 11
      }
    },
    yAxis: [
      {
        type: 'value',
        name: '分割次数',
        position: 'left',
        axisLine: {
          lineStyle: {
            color: '#E6A23C'
          }
        },
        axisLabel: {
          color: '#909399'
        },
        splitLine: {
          lineStyle: {
            color: '#f5f7fa'
          }
        }
      },
      {
        type: 'value',
        name: '平均置信度',
        position: 'right',
        min: 0.5,
        max: 1,
        axisLine: {
          lineStyle: {
            color: '#F56C6C'
          }
        },
        axisLabel: {
          color: '#909399',
          formatter: '{value}'
        }
      }
    ],
    series: [
      {
        name: '分割次数',
        type: 'bar',
        data: statsData.value.patient_stats.slice(0, 10).map(item => item.segmentation_count),
        itemStyle: {
          color: '#E6A23C'
        },
        barWidth: '50%'
      },
      {
        name: '平均置信度',
        type: 'line',
        yAxisIndex: 1,
        data: statsData.value.patient_stats.slice(0, 10).map(item => Number(item.avg_confidence.toFixed(3))),
        itemStyle: {
          color: '#F56C6C'
        },
        lineStyle: {
          width: 3
        },
        symbol: 'circle',
        symbolSize: 6
      }
    ]
  }
}

onMounted(() => {
  fetchStats()
})
</script>

<template>
  <div class="dashboard-container">
    <!-- 顶部标题栏 -->
    <div class="header-section">
      <div class="header-content">
        <div class="title-area">
          <h1 class="dashboard-title">🛣️ 道路缺陷检测分析中心</h1>
          <p class="dashboard-subtitle">道路缺陷检测模型性能监控与数据分析</p>
        </div>
        <div class="action-area">
          <el-button type="primary" :icon="Refresh" :loading="loading" size="large" @click="fetchStats">
            刷新数据
          </el-button>
        </div>
      </div>
    </div>

    <!-- 错误提示 -->
    <el-alert v-if="error" :title="error" type="error" class="error-alert" show-icon />

    <!-- 主要内容区域 -->
    <div class="main-content">
      <!-- 左侧统计面板 -->
      <div class="stats-sidebar">
        <div class="sidebar-header">
          <h3 class="sidebar-title">📊 核心指标</h3>
        </div>
        
        <div v-if="statsData" class="stats-list">
          <div v-for="(stat, index) in statsCards" :key="stat.key" class="stat-item" :class="stat.colorClass">
            <div class="stat-icon">
              <el-icon :size="24">
                <component :is="stat.icon" />
              </el-icon>
            </div>
            <div class="stat-content">
              <div class="stat-label">{{ stat.label }}</div>
              <div class="stat-value">{{ stat.value }}</div>
              <div class="stat-trend" v-if="stat.trend">
                <span class="trend-text">{{ stat.trend }}</span>
              </div>
            </div>
          </div>
        </div>

        <!-- 加载状态 -->
        <div v-else-if="loading" class="stats-loading">
          <div v-for="i in 4" :key="i" class="stat-skeleton">
            <div class="skeleton-icon"></div>
            <div class="skeleton-content">
              <div class="skeleton-line short"></div>
              <div class="skeleton-line long"></div>
            </div>
          </div>
        </div>
      </div>

      <!-- 右侧图表区域 -->
      <div class="charts-main">
        <div v-if="statsData && !loading" class="charts-container">
          <!-- 图表标签页 -->
          <el-tabs v-model="activeTab" class="chart-tabs" type="border-card">
            <el-tab-pane label="📈 趋势分析" name="trend">
              <div class="chart-wrapper">
                <div class="chart-header">
                  <h4 class="chart-title">每日分割统计趋势</h4>
                  <p class="chart-desc">展示最近30天的模型使用情况和性能变化</p>
                </div>
                <div class="chart-content">
                  <v-chart :option="dailyStatsOption" autoresize class="chart" />
                </div>
              </div>
            </el-tab-pane>
            
            <el-tab-pane label="🎯 置信度分析" name="confidence">
              <div class="chart-wrapper">
                <div class="chart-header">
                  <h4 class="chart-title">模型置信度分布</h4>
                  <p class="chart-desc">分析模型预测结果的置信度分布情况</p>
                </div>
                <div class="chart-content">
                  <v-chart :option="confidenceDistributionOption" autoresize class="chart" />
                </div>
              </div>
            </el-tab-pane>
            
            <el-tab-pane label="👥 患者统计" name="patient">
              <div class="chart-wrapper">
                <div class="chart-header">
                  <h4 class="chart-title">患者分割数据统计</h4>
                  <p class="chart-desc">各患者的模型使用频次和平均性能指标</p>
                </div>
                <div class="chart-content">
                  <v-chart :option="patientStatsOption" autoresize class="chart" />
                </div>
              </div>
            </el-tab-pane>
          </el-tabs>
        </div>

        <!-- 图表加载状态 -->
        <div v-else-if="loading" class="charts-loading">
          <div class="loading-content">
            <el-icon class="loading-icon" size="64">
              <Refresh />
            </el-icon>
            <p class="loading-text">正在加载数据分析...</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.dashboard-container {
  min-height: 100vh;
  background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
  padding: 0;
}

/* 顶部标题栏 */
.header-section {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  padding: 2rem 0;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1);
}

.header-content {
  max-width: 1400px;
  margin: 0 auto;
  padding: 0 2rem;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.title-area {
  flex: 1;
}

.dashboard-title {
  font-size: 2.5rem;
  font-weight: 700;
  margin: 0 0 0.5rem 0;
  text-shadow: 0 2px 4px rgba(0, 0, 0, 0.3);
}

.dashboard-subtitle {
  font-size: 1.1rem;
  opacity: 0.9;
  margin: 0;
  font-weight: 300;
}

.action-area {
  flex-shrink: 0;
}

/* 错误提示 */
.error-alert {
  max-width: 1400px;
  margin: 1rem auto;
  margin-left: 2rem;
  margin-right: 2rem;
}

/* 主要内容区域 */
.main-content {
  max-width: 1400px;
  margin: 0 auto;
  padding: 2rem;
  display: grid;
  grid-template-columns: 320px 1fr;
  gap: 2rem;
  align-items: start;
}

/* 左侧统计面板 */
.stats-sidebar {
  background: white;
  border-radius: 16px;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.08);
  overflow: hidden;
  position: sticky;
  top: 2rem;
  animation: slideInLeft 0.8s ease-out;
}

.stats-sidebar:hover {
  transform: translateY(-3px);
  box-shadow: 0 12px 40px rgba(0, 0, 0, 0.12);
}

.sidebar-header {
  background: linear-gradient(135deg, #4f46e5 0%, #7c3aed 100%);
  color: white;
  padding: 1.5rem;
  text-align: center;
}

.sidebar-title {
  margin: 0;
  font-size: 1.2rem;
  font-weight: 600;
}

.stats-list {
  padding: 1rem;
}

.stat-item {
  display: flex;
  align-items: center;
  padding: 1.2rem;
  margin-bottom: 0.8rem;
  border-radius: 12px;
  transition: all 0.3s ease;
  cursor: pointer;
  border-left: 4px solid transparent;
  animation: slideInUp 0.6s ease-out;
}

.stat-item:hover {
  transform: translateX(4px) translateY(-2px);
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.15);
}

.stat-item:last-child {
  margin-bottom: 0;
}

.stat-blue {
  background: linear-gradient(135deg, #dbeafe 0%, #bfdbfe 100%);
  border-left-color: #3b82f6;
}

.stat-green {
  background: linear-gradient(135deg, #dcfce7 0%, #bbf7d0 100%);
  border-left-color: #10b981;
}

.stat-orange {
  background: linear-gradient(135deg, #fed7aa 0%, #fdba74 100%);
  border-left-color: #f59e0b;
}

.stat-purple {
  background: linear-gradient(135deg, #e9d5ff 0%, #d8b4fe 100%);
  border-left-color: #8b5cf6;
}

.stat-icon {
  width: 48px;
  height: 48px;
  border-radius: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-right: 1rem;
  background: rgba(255, 255, 255, 0.8);
  color: #4b5563;
  transition: all 0.3s ease;
}

.stat-item:hover .stat-icon {
  transform: rotate(10deg) scale(1.1);
}

.stat-content {
  flex: 1;
}

.stat-label {
  font-size: 0.875rem;
  color: #6b7280;
  margin-bottom: 0.25rem;
  font-weight: 500;
}

.stat-value {
  font-size: 1.75rem;
  font-weight: 700;
  color: #1f2937;
  margin-bottom: 0.25rem;
}

.stat-trend {
  font-size: 0.75rem;
  color: #059669;
  font-weight: 500;
}

.trend-text {
  background: rgba(16, 185, 129, 0.1);
  padding: 0.2rem 0.5rem;
  border-radius: 6px;
}

/* 加载状态 */
.stats-loading {
  padding: 1rem;
}

.stat-skeleton {
  display: flex;
  align-items: center;
  padding: 1.2rem;
  margin-bottom: 0.8rem;
  border-radius: 12px;
  background: #f3f4f6;
}

.skeleton-icon {
  width: 48px;
  height: 48px;
  border-radius: 12px;
  background: #e5e7eb;
  margin-right: 1rem;
  animation: pulse 2s infinite;
}

.skeleton-content {
  flex: 1;
}

.skeleton-line {
  height: 12px;
  background: #e5e7eb;
  border-radius: 6px;
  margin-bottom: 0.5rem;
  animation: pulse 2s infinite;
}

.skeleton-line.short {
  width: 60%;
}

.skeleton-line.long {
  width: 80%;
}

@keyframes pulse {
  0%, 100% {
    opacity: 1;
  }
  50% {
    opacity: 0.5;
  }
}

/* 右侧图表区域 */
.charts-main {
  background: white;
  border-radius: 16px;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.08);
  overflow: hidden;
  animation: slideInRight 0.8s ease-out;
}

.charts-main:hover {
  transform: translateY(-3px);
  box-shadow: 0 12px 40px rgba(0, 0, 0, 0.12);
}

.charts-container {
  height: 100%;
  min-height: 700px;
}

.chart-tabs {
  height: 100%;
}

.chart-tabs :deep(.el-tabs__content) {
  height: calc(100% - 60px);
  min-height: 540px;
  padding: 0;
}

.chart-tabs :deep(.el-tab-pane) {
  height: 100%;
  min-height: 600px;
}

.chart-wrapper {
  padding: 2rem;
  height: 100%;
  display: flex;
  flex-direction: column;
}

.chart-header {
  margin-bottom: 1.5rem;
}

.chart-title {
  font-size: 1.5rem;
  font-weight: 600;
  color: #1f2937;
  margin: 0 0 0.5rem 0;
}

.chart-desc {
  font-size: 0.875rem;
  color: #6b7280;
  margin: 0;
}

.chart-content {
  flex: 1;
  min-height: 500px;
  height: 500px;
}

.chart {
  width: 100%;
  height: 100%;
  min-height: 500px;
}

/* 图表加载状态 */
.charts-loading {
  height: 600px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.loading-content {
  text-align: center;
}

.loading-icon {
  color: #6b7280;
  animation: spin 2s linear infinite;
  margin-bottom: 1rem;
}

.loading-text {
  font-size: 1.1rem;
  color: #6b7280;
  margin: 0;
}

@keyframes spin {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}

@keyframes slideInLeft {
  from {
    opacity: 0;
    transform: translateX(-50px);
  }
  to {
    opacity: 1;
    transform: translateX(0);
  }
}

@keyframes slideInRight {
  from {
    opacity: 0;
    transform: translateX(50px);
  }
  to {
    opacity: 1;
    transform: translateX(0);
  }
}

@keyframes slideInUp {
  from {
    opacity: 0;
    transform: translateY(30px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

/* 为统计项添加延迟动画 */
.stat-item:nth-child(1) { animation-delay: 0.1s; }
.stat-item:nth-child(2) { animation-delay: 0.2s; }
.stat-item:nth-child(3) { animation-delay: 0.3s; }
.stat-item:nth-child(4) { animation-delay: 0.4s; }

/* 响应式设计 */
@media (max-width: 1200px) {
  .main-content {
    grid-template-columns: 280px 1fr;
    gap: 1.5rem;
  }
  
  .dashboard-title {
    font-size: 2rem;
  }
}

@media (max-width: 968px) {
  .main-content {
    grid-template-columns: 1fr;
    gap: 1.5rem;
  }
  
  .stats-sidebar {
    position: static;
  }
  
  .header-content {
    flex-direction: column;
    text-align: center;
    gap: 1rem;
  }
  
  .dashboard-title {
    font-size: 1.8rem;
  }
}

@media (max-width: 640px) {
  .dashboard-container {
    padding: 0;
  }
  
  .header-content {
    padding: 0 1rem;
  }
  
  .main-content {
    padding: 1rem;
  }
  
  .chart-wrapper {
    padding: 1rem;
  }
  
  .dashboard-title {
    font-size: 1.5rem;
  }
  
  .chart-content {
    min-height: 400px;
  }
}
</style>