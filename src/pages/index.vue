<template>
  <div class="bg-base-100 text-base-content flex flex-col overflow-hidden">
    <!-- 主内容区 -->
    <div class="flex-1">
      <!-- 内容区域 -->
      <main class="flex-1 overflow-auto p-6">
        <div class="space-y-8">
          <!-- 欢迎区域 -->
          <div class="bg-base-200 p-8 rounded-lg shadow-md text-center">
            <h2 class="text-3xl font-bold mb-4">欢迎使用道路缺陷检测系统</h2>
            <p class="text-lg opacity-80">一站式智能检测与分析平台，助您高效管理道路养护。</p>
            <div class="mt-6">
              <button class="btn btn-primary btn-lg">立即开始检测</button>
            </div>
          </div>

          <!-- 核心功能概览 -->
          <div>
            <h3 class="text-2xl font-semibold mb-4">核心功能</h3>
            <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
              <div v-for="(f, i) in features" :key="i" class="card bg-base-200 shadow-sm">
                <div class="card-body">
                  <h4 class="card-title">{{ f.title }}</h4>
                  <p class="text-sm opacity-70">{{ f.desc }}</p>
                </div>
              </div>
            </div>
          </div>

          <!-- 快速入口 -->
          <div>
            <h3 class="text-2xl font-semibold mb-4">快速入口</h3>
            <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
              <router-link to="/history" class="btn btn-outline btn-lg">查看检测记录</router-link>
              <router-link to="/about" class="btn btn-outline btn-lg">关于本系统</router-link>
            </div>
          </div>
        </div>
      </main>
      
      <!-- 页脚 -->
      <footer class="bg-base-100 border-t border-base-300 p-4 text-center text-sm opacity-70">
        {{ footerText }}
      </footer>
    </div>
  </div>
</template>

<script setup lang="ts">
/**
 * 动画说明：
 * - 使用你指定的 motion-v：import { Motion } from 'motion-v'
 * - RevealMotion 组件内部用 IntersectionObserver 监听进入视口
 * - 初始：opacity 0 + y 16；进入：opacity 1 + y 0
 */
import { ref, computed } from "vue";

const footerText = import.meta.env.VITE_APP_FOOTER || "版权所有 © 2025 HZSYSTEM";

// 道路缺陷检测系统功能特性
const features = [
  { title: "智能检测", desc: "利用先进的深度学习算法，精准识别各类道路缺陷。" },
  { title: "实时分析", desc: "快速处理上传的道路图像，即时生成详细分析报告。" },
  { title: "数据统计", desc: "提供全面的检测历史记录和多维度统计分析，辅助决策。" },
  { title: "模型管理", desc: "灵活配置和优化检测模型，确保系统性能始终处于最佳状态。" },
  { title: "可视化结果", desc: "直观展示缺陷标注和叠加图像，便于用户快速理解检测结果。" },
  { title: "辅助决策", desc: "提供缺陷类型、位置、严重程度等关键指标，为道路养护提供科学依据。" },
];
</script>

<style scoped>
/* 禁用滚动 */
html,
body {
  overflow: hidden;
}

</style>
