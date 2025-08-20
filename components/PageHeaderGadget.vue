<template>
  <div class="page-header-gadget">
    <div class="header-content">
      <div class="company-logo">
        <img
          v-if="logoUrl"
          :src="logoUrl"
          alt="Company Logo"
          class="logo-img"
        >
        <div v-else class="logo-placeholder">
          LOGO
        </div>
      </div>
      <div class="header-info">
        <h1 class="report-title">
          {{ title || 'Business Report' }}
        </h1>
        <div class="report-meta">
          <span class="report-date">{{ formatDate(new Date()) }}</span>
          <span class="report-type">Dashboard Report</span>
        </div>
      </div>
      <div class="header-actions">
        <div class="export-options">
          <button class="export-btn">
            <i class="icon-download" />
            Export
          </button>
        </div>
      </div>
    </div>
    <div class="header-divider" />
  </div>
</template>

<script>
export default {
  name: 'PageHeaderGadget',
  props: {
    item: {
      type: Object,
      required: true
    },
    title: {
      type: String,
      default: 'Business Report'
    },
    logoUrl: {
      type: String,
      default: ''
    }
  },
  methods: {
    formatDate (date) {
      return date.toLocaleDateString('en-US', {
        year: 'numeric',
        month: 'long',
        day: 'numeric'
      })
    }
  }
}
</script>

<style scoped>
.page-header-gadget {
  width: 100%;
  height: 100%;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  box-sizing: border-box;
  overflow: hidden;
}

.header-content {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 15px;
}

.company-logo {
  flex-shrink: 0;
}

.logo-img {
  width: 60px;
  height: 60px;
  object-fit: contain;
  border-radius: 8px;
  background: rgba(255, 255, 255, 0.1);
  padding: 8px;
}

.logo-placeholder {
  width: 60px;
  height: 60px;
  background: rgba(255, 255, 255, 0.2);
  border: 2px dashed rgba(255, 255, 255, 0.5);
  border-radius: 8px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 12px;
  font-weight: bold;
}

.header-info {
  flex: 1;
  text-align: center;
  margin: 0 20px;
}

.report-title {
  font-size: 28px;
  font-weight: bold;
  margin: 0 0 8px 0;
  text-shadow: 0 2px 4px rgba(0, 0, 0, 0.3);
}

.report-meta {
  display: flex;
  justify-content: center;
  gap: 20px;
  font-size: 14px;
  opacity: 0.9;
}

.header-actions {
  flex-shrink: 0;
}

.export-btn {
  background: rgba(255, 255, 255, 0.2);
  border: 1px solid rgba(255, 255, 255, 0.3);
  color: white;
  padding: 8px 16px;
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.3s;
  display: flex;
  align-items: center;
  gap: 8px;
}

.export-btn:hover {
  background: rgba(255, 255, 255, 0.3);
}

.icon-download::before {
  content: '⬇';
}

.header-divider {
  height: 3px;
  background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.5), transparent);
  border-radius: 2px;
}

@media (max-width: 768px) {
  .header-content {
    flex-direction: column;
    gap: 15px;
    text-align: center;
  }

  .header-info {
    margin: 0;
  }

  .report-title {
    font-size: 24px;
  }

  .report-meta {
    flex-direction: column;
    gap: 5px;
  }
}
</style>
