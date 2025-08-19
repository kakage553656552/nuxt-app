<template>
  <div>
    <publicPdf
      :layout="layout"
      :col-num="12"
      :row-height="60"
      :is-draggable="true"
      :is-resizable="true"
      :vertical-compact="true"
      :margin="[10, 10]"
      :use-css-transforms="true"
      @layout-updated="onLayoutUpdated"
    >
      <template #default="{ item }">
        <div class="custom-widget">
          <div class="widget-header">
            <h4>{{ item.title }}</h4>
            <button class="remove-btn" @click="removeWidget(item.i)">
              ×
            </button>
          </div>
          <div class="widget-body">
            <p>{{ item.content }}</p>
          </div>
        </div>
      </template>
    </publicPdf>

    <div class="control-panel">
      <el-button type="primary" @click="addWidget">
        添加Widget
      </el-button>
      <el-button type="success" @click="saveLayout">
        保存布局
      </el-button>
      <el-button type="info" @click="resetLayout">
        重置布局
      </el-button>
    </div>
  </div>
</template>

<script>
export default {
  name: 'IndexPage',
  data () {
    return {
      layout: [
        { x: 0, y: 0, w: 2, h: 2, i: '0', title: 'Widget 1', content: '这是第一个可拖拽的Widget组件' },
        { x: 2, y: 0, w: 4, h: 3, i: '1', title: 'Widget 2', content: '这是第二个Widget，可以调整大小和位置' },
        { x: 6, y: 0, w: 3, h: 2, i: '2', title: 'Widget 3', content: '第三个Widget，支持响应式布局' }
      ],
      nextWidgetId: 3
    }
  },
  mounted () {
    const savedLayout = localStorage.getItem('vueGridLayout')
    if (savedLayout) {
      this.layout = JSON.parse(savedLayout)
      this.nextWidgetId = Math.max(...this.layout.map(item => parseInt(item.i))) + 1
    }
  },
  methods: {
    onLayoutUpdated (newLayout) {
      this.layout = newLayout
    },

    addWidget () {
      const newWidget = {
        x: 0,
        y: 0,
        w: 2,
        h: 2,
        i: this.nextWidgetId.toString(),
        title: `Widget ${this.nextWidgetId + 1}`,
        content: `新添加的Widget ${this.nextWidgetId + 1}`
      }
      this.layout.push(newWidget)
      this.nextWidgetId++
    },

    removeWidget (widgetId) {
      this.layout = this.layout.filter(item => item.i !== widgetId)
    },

    saveLayout () {
      localStorage.setItem('vueGridLayout', JSON.stringify(this.layout))
      this.$message.success('布局已保存')
    },

    resetLayout () {
      this.layout = [
        { x: 0, y: 0, w: 2, h: 2, i: '0', title: 'Widget 1', content: '这是第一个可拖拽的Widget组件' },
        { x: 2, y: 0, w: 4, h: 3, i: '1', title: 'Widget 2', content: '这是第二个Widget，可以调整大小和位置' },
        { x: 6, y: 0, w: 3, h: 2, i: '2', title: 'Widget 3', content: '第三个Widget，支持响应式布局' }
      ]
      this.nextWidgetId = 3
    }
  }
}
</script>

<style scoped>
.control-panel {
  margin-top: 20px;
  text-align: center;
}

.control-panel .el-button {
  margin: 0 10px;
}

.custom-widget {
  height: 100%;
  display: flex;
  flex-direction: column;
}

.widget-header {
  background: #409eff;
  color: white;
  padding: 8px 12px;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.widget-header h4 {
  margin: 0;
  font-size: 14px;
}

.remove-btn {
  background: transparent;
  border: none;
  color: white;
  font-size: 18px;
  cursor: pointer;
  padding: 0;
  width: 20px;
  height: 20px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
}

.remove-btn:hover {
  background: rgba(255, 255, 255, 0.2);
}

.widget-body {
  flex: 1;
  padding: 12px;
  font-size: 12px;
  color: #666;
}
</style>
