<template>
  <div class="card">
    <div class="card-header">
      <div>
        <p class="eyebrow">交付物</p>
        <h2>导出报告</h2>
      </div>
      <div v-if="!isAIUsed" class="ai-status-badge">
        <span class="ai-status-icon">📋</span>
        <span class="ai-status-text">未使用AI助手</span>
      </div>
    </div>

    <div v-if="!isAIUsed" class="ai-notice">
      当前报告未使用AI助手生成，AI推荐理由将不会显示在报告中。
    </div>

    <div class="button-group">
      <el-button type="primary" plain @click="$emit('export')" :loading="exporting">
        导出比价报告
      </el-button>
      <el-button
        v-if="report"
        type="success"
        plain
        @click="$emit('download-word')"
        :loading="downloadingWord"
      >
        生成为Word文档
      </el-button>
    </div>

    <pre v-if="report" class="report-content">{{ report }}</pre>
  </div>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  report: { type: String, default: '' },
  exporting: { type: Boolean, default: false },
  downloadingWord: { type: Boolean, default: false },
  connectionStatus: {
    type: Object,
    default: () => ({ online: true, llm: false, error: '' })
  },
  agentSnapshot: {
    type: Object,
    default: null
  }
})

defineEmits(['export', 'download-word'])

// 判断是否使用了AI助手
const isAIUsed = computed(() => {
  // 如果网络或LLM未连接，则未使用AI
  if (!props.connectionStatus.online || !props.connectionStatus.llm) {
    return false
  }
  // 如果有agentSnapshot且parse_source是offline_regex，则未使用AI
  if (props.agentSnapshot?.parse_source === 'offline_regex') {
    return false
  }
  // 如果feedback_source是template或null，则未使用AI生成反馈
  if (props.agentSnapshot?.feedback_source === 'template' ||
      props.agentSnapshot?.feedback_source === null) {
    return false
  }
  return true
})
</script>

<style scoped>
.card {
  background: white;
  border: 1px solid #d9e2ec;
  border-radius: 8px;
  padding: 18px;
  box-shadow: 0 12px 30px rgba(15, 23, 42, 0.05);
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  margin-bottom: 14px;
}

.card h2 {
  color: #1f2937;
  font-size: 18px;
}

.eyebrow {
  color: #64748b;
  font-size: 12px;
  font-weight: 700;
  margin-bottom: 4px;
}

.ai-status-badge {
  display: flex;
  align-items: center;
  gap: 6px;
  background: #fef3c7;
  border: 1px solid #fbbf24;
  border-radius: 6px;
  padding: 6px 12px;
}

.ai-status-icon {
  font-size: 14px;
}

.ai-status-text {
  font-size: 12px;
  color: #92400e;
  font-weight: 600;
}

.ai-notice {
  background: #fff7ed;
  border: 1px solid #fed7aa;
  border-radius: 6px;
  padding: 10px 14px;
  margin-bottom: 14px;
  font-size: 13px;
  color: #c2410c;
}

.button-group {
  display: flex;
  gap: 10px;
  flex-wrap: wrap;
}

.report-content {
  background: #f8fafc;
  border: 1px solid #e2e8f0;
  padding: 20px;
  border-radius: 6px;
  margin-top: 15px;
  white-space: pre-wrap;
  font-size: 13px;
}
</style>
