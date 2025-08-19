<template>
  <div class="card-number-gadget">
    <div class="card-header">
      <div class="card-title">
        <h3>{{ item.name || 'Metric Card' }}</h3>
        <div v-if="integrationInfo" class="integration-info">
          <img
            :src="integrationInfo.icon"
            :alt="integrationInfo.name"
            class="integration-icon"
          >
          <span class="integration-name">{{ integrationInfo.name }}</span>
        </div>
      </div>
      <div class="card-actions">
        <button class="action-btn">
          ⋯
        </button>
      </div>
    </div>

    <div class="card-content">
      <div class="metric-display">
        <div class="metric-value">
          {{ formatNumber(metricValue) }}
        </div>
        <div class="metric-label">
          {{ item.name || 'Total Count' }}
        </div>
      </div>

      <div class="metric-trend">
        <div
          class="trend-indicator"
          :class="trendClass"
        >
          <span class="trend-icon">{{ trendIcon }}</span>
          <span class="trend-value">{{ trendPercentage }}%</span>
        </div>
        <div class="trend-label">
          vs last period
        </div>
      </div>
    </div>

    <div class="card-footer">
      <div class="business-type">
        <span class="type-badge">{{ item.businessType || 'Standard' }}</span>
      </div>
      <div class="last-updated">
        Updated {{ formatTime(lastUpdated) }}
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'CardNumberGadget',
  props: {
    item: {
      type: Object,
      required: true
    }
  },
  data () {
    return {
      metricValue: this.generateRandomValue(),
      trendPercentage: this.generateRandomTrend(),
      lastUpdated: new Date()
    }
  },
  computed: {
    integrationInfo () {
      if (this.item.integrationList && this.item.integrationList.length > 0) {
        return this.item.integrationList[0]
      }
      return null
    },
    trendClass () {
      return this.trendPercentage >= 0 ? 'trend-up' : 'trend-down'
    },
    trendIcon () {
      return this.trendPercentage >= 0 ? '↗' : '↘'
    }
  },
  methods: {
    generateRandomValue () {
      return Math.floor(Math.random() * 1000) + 100
    },
    generateRandomTrend () {
      return (Math.random() - 0.5) * 40
    },
    formatNumber (value) {
      return value.toLocaleString()
    },
    formatTime (date) {
      const now = new Date()
      const diffMs = now - date
      const diffMins = Math.floor(diffMs / 60000)

      if (diffMins < 1) { return 'just now' }
      if (diffMins < 60) { return `${diffMins}m ago` }
      if (diffMins < 1440) { return `${Math.floor(diffMins / 60)}h ago` }
      return `${Math.floor(diffMins / 1440)}d ago`
    }
  }
}
</script>

<style scoped>
.card-number-gadget {
  background: white;
  border-radius: 12px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  border: 1px solid #e5e7eb;
  overflow: hidden;
  transition: all 0.3s ease;
  height: 100%;
  display: flex;
  flex-direction: column;
}

.card-number-gadget:hover {
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.15);
  transform: translateY(-2px);
}

.card-header {
  padding: 16px 20px;
  border-bottom: 1px solid #f3f4f6;
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
}

.card-title h3 {
  margin: 0 0 8px 0;
  font-size: 16px;
  font-weight: 600;
  color: #1f2937;
}

.integration-info {
  display: flex;
  align-items: center;
  gap: 8px;
}

.integration-icon {
  width: 20px;
  height: 20px;
  border-radius: 4px;
}

.integration-name {
  font-size: 12px;
  color: #6b7280;
}

.action-btn {
  background: none;
  border: none;
  color: #9ca3af;
  cursor: pointer;
  padding: 4px;
  border-radius: 4px;
  transition: background-color 0.2s;
}

.action-btn:hover {
  background: #f3f4f6;
}

.card-content {
  flex: 1;
  padding: 20px;
  display: flex;
  flex-direction: column;
  justify-content: center;
}

.metric-display {
  text-align: center;
  margin-bottom: 20px;
}

.metric-value {
  font-size: 36px;
  font-weight: bold;
  color: #1f2937;
  line-height: 1;
  margin-bottom: 8px;
}

.metric-label {
  font-size: 14px;
  color: #6b7280;
  font-weight: 500;
}

.metric-trend {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 12px;
}

.trend-indicator {
  display: flex;
  align-items: center;
  gap: 4px;
  padding: 4px 8px;
  border-radius: 6px;
  font-size: 14px;
  font-weight: 600;
}

.trend-up {
  background: #d1fae5;
  color: #065f46;
}

.trend-down {
  background: #fee2e2;
  color: #991b1b;
}

.trend-label {
  font-size: 12px;
  color: #9ca3af;
}

.card-footer {
  padding: 12px 20px;
  background: #f9fafb;
  border-top: 1px solid #f3f4f6;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.type-badge {
  background: #e0e7ff;
  color: #3730a3;
  padding: 2px 8px;
  border-radius: 12px;
  font-size: 11px;
  font-weight: 600;
  text-transform: uppercase;
}

.last-updated {
  font-size: 11px;
  color: #9ca3af;
}

@media (max-width: 480px) {
  .metric-value {
    font-size: 28px;
  }

  .card-content {
    padding: 16px;
  }

  .card-header {
    padding: 12px 16px;
  }

  .card-footer {
    padding: 8px 16px;
    flex-direction: column;
    gap: 4px;
    align-items: flex-start;
  }
}
</style>
