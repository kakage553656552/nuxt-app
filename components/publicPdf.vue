<template>
  <div class="vue-grid-layout">
    <div
      v-for="item in layout"
      :key="item.i"
      :style="getItemStyle(item)"
      class="vue-grid-item"
      :class="{
        'vue-grid-item--dragging': isDragging && draggedItem && draggedItem.i === item.i,
        'vue-grid-item--resizing': isResizing && resizingItem && resizingItem.i === item.i
      }"
      :draggable="isDraggable"
      @dragstart="onDragStart($event, item)"
      @dragend="onDragEnd"
    >
      <div class="vue-grid-item-content">
        <slot
          :item="item"
          :index="layout.indexOf(item)"
        >
          <div class="default-widget">
            <div class="widget-header">
              <h4>{{ item.title || `Item ${item.i}` }}</h4>
              <button
                v-if="isResizable"
                class="resize-handle"
                @mousedown="startResize($event, item)"
              >
                ⤡
              </button>
            </div>
            <div class="widget-body">
              <p>{{ item.content || `Widget content ${item.i}` }}</p>
            </div>
          </div>
        </slot>
      </div>
    </div>

    <div
      v-if="dragPlaceholder.visible"
      :style="dragPlaceholder.style"
      class="vue-grid-placeholder"
    />
  </div>
</template>

<script>
export default {
  name: 'PublicPdf',
  props: {
    layout: {
      type: Array,
      required: true
    },
    colNum: {
      type: Number,
      default: 12
    },
    rowHeight: {
      type: Number,
      default: 60
    },
    isDraggable: {
      type: Boolean,
      default: true
    },
    isResizable: {
      type: Boolean,
      default: true
    },
    isMirrored: {
      type: Boolean,
      default: false
    },
    verticalCompact: {
      type: Boolean,
      default: true
    },
    margin: {
      type: Array,
      default: () => [10, 10]
    },
    useCssTransforms: {
      type: Boolean,
      default: true
    },
    responsive: {
      type: Boolean,
      default: false
    },
    responsiveLayouts: {
      type: Object,
      default: () => ({})
    },
    breakpoints: {
      type: Object,
      default: () => ({ lg: 1200, md: 996, sm: 768, xs: 480, xxs: 0 })
    },
    cols: {
      type: Object,
      default: () => ({ lg: 12, md: 10, sm: 6, xs: 4, xxs: 2 })
    }
  },
  data () {
    return {
      containerWidth: 0,
      isDragging: false,
      isResizing: false,
      draggedItem: null,
      resizingItem: null,
      dragOffset: { x: 0, y: 0 },
      resizeStartPos: { x: 0, y: 0 },
      resizeStartSize: { w: 0, h: 0 },
      dragPlaceholder: {
        visible: false,
        style: {}
      }
    }
  },
  computed: {
    colWidth () {
      return (this.containerWidth - (this.colNum + 1) * this.margin[0]) / this.colNum
    },
    currentBreakpoint () {
      if (!this.responsive) { return 'lg' }
      const width = this.containerWidth
      const breakpoints = this.breakpoints
      return Object.keys(breakpoints)
        .sort((a, b) => breakpoints[b] - breakpoints[a])
        .find(bp => width >= breakpoints[bp]) || 'xxs'
    },
    currentCols () {
      return this.responsive ? this.cols[this.currentBreakpoint] : this.colNum
    }
  },
  mounted () {
    this.updateContainerWidth()
    window.addEventListener('resize', this.updateContainerWidth)
    document.addEventListener('mousemove', this.onMouseMove)
    document.addEventListener('mouseup', this.onMouseUp)
  },
  beforeDestroy () {
    window.removeEventListener('resize', this.updateContainerWidth)
    document.removeEventListener('mousemove', this.onMouseMove)
    document.removeEventListener('mouseup', this.onMouseUp)
  },
  methods: {
    updateContainerWidth () {
      if (this.$el) {
        this.containerWidth = this.$el.offsetWidth
      }
    },

    getItemStyle (item) {
      const colWidth = this.colWidth
      const rowHeight = this.rowHeight
      const margin = this.margin

      const x = item.x * colWidth + (item.x + 1) * margin[0]
      const y = item.y * rowHeight + (item.y + 1) * margin[1]
      const width = item.w * colWidth + (item.w - 1) * margin[0]
      const height = item.h * rowHeight + (item.h - 1) * margin[1]

      const style = {
        left: x + 'px',
        top: y + 'px',
        width: width + 'px',
        height: height + 'px',
        position: 'absolute'
      }

      if (this.useCssTransforms) {
        style.transform = `translate3d(${x}px, ${y}px, 0)`
        style.left = '0'
        style.top = '0'
      }

      return style
    },

    onDragStart (event, item) {
      if (!this.isDraggable) {
        return
      }

      this.isDragging = true
      this.draggedItem = item
      const rect = event.target.getBoundingClientRect()

      this.dragOffset = {
        x: event.clientX - rect.left,
        y: event.clientY - rect.top
      }

      event.dataTransfer.effectAllowed = 'move'
      event.dataTransfer.setData('text/html', '')

      this.dragPlaceholder.visible = true
      this.updatePlaceholder(item)

      this.$emit('layout-ready', this.layout)
      this.$emit('layout-before-mount', this.layout)
      this.$emit('layout-mounted', this.layout)
    },

    onDragEnd () {
      this.isDragging = false
      this.draggedItem = null
      this.dragPlaceholder.visible = false
      this.$emit('layout-updated', this.layout)
    },

    startResize (event, item) {
      if (!this.isResizable) {
        return
      }

      event.preventDefault()
      event.stopPropagation()

      this.isResizing = true
      this.resizingItem = item
      this.resizeStartPos = { x: event.clientX, y: event.clientY }
      this.resizeStartSize = { w: item.w, h: item.h }
    },

    onMouseMove (event) {
      if (this.isDragging && this.draggedItem) {
        this.updateDraggedItemPosition(event)
      } else if (this.isResizing && this.resizingItem) {
        this.updateResizingItemSize(event)
      }
    },

    onMouseUp () {
      if (this.isResizing) {
        this.isResizing = false
        this.resizingItem = null
        this.$emit('layout-updated', this.layout)
      }
    },

    updateDraggedItemPosition (event) {
      const containerRect = this.$el.getBoundingClientRect()
      const colWidth = this.colWidth
      const rowHeight = this.rowHeight
      const margin = this.margin

      const newX = Math.max(0, Math.round((event.clientX - containerRect.left - this.dragOffset.x) / (colWidth + margin[0])))
      const newY = Math.max(0, Math.round((event.clientY - containerRect.top - this.dragOffset.y) / (rowHeight + margin[1])))

      if (newX !== this.draggedItem.x || newY !== this.draggedItem.y) {
        this.draggedItem.x = Math.min(newX, this.currentCols - this.draggedItem.w)
        this.draggedItem.y = newY
        this.updatePlaceholder(this.draggedItem)
      }
    },

    updateResizingItemSize (event) {
      const deltaX = event.clientX - this.resizeStartPos.x
      const deltaY = event.clientY - this.resizeStartPos.y
      const colWidth = this.colWidth
      const rowHeight = this.rowHeight
      const margin = this.margin

      const newW = Math.max(1, this.resizeStartSize.w + Math.round(deltaX / (colWidth + margin[0])))
      const newH = Math.max(1, this.resizeStartSize.h + Math.round(deltaY / (rowHeight + margin[1])))

      this.resizingItem.w = Math.min(newW, this.currentCols - this.resizingItem.x)
      this.resizingItem.h = newH
    },

    updatePlaceholder (item) {
      this.dragPlaceholder.style = this.getItemStyle(item)
    },

    // 公开的API方法
    addItem (item) {
      const newLayout = [...this.layout, item]
      this.$emit('layout-updated', newLayout)
    },

    removeItem (itemId) {
      const newLayout = this.layout.filter(item => item.i !== itemId)
      this.$emit('layout-updated', newLayout)
    },

    moveItem (itemId, x, y) {
      const newLayout = this.layout.map((item) => {
        if (item.i === itemId) {
          return { ...item, x, y }
        }
        return item
      })
      this.$emit('layout-updated', newLayout)
    },

    resizeItem (itemId, w, h) {
      const newLayout = this.layout.map((item) => {
        if (item.i === itemId) {
          return { ...item, w, h }
        }
        return item
      })
      this.$emit('layout-updated', newLayout)
    }
  }
}
</script>

<style scoped>
.vue-grid-layout {
  position: relative;
  background: #eee;
  border-radius: 8px;
  padding: 10px;
  min-height: 400px;
}

.vue-grid-item {
  background: white;
  border-radius: 6px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  border: 1px solid #ddd;
  transition: all 0.2s ease;
  cursor: move;
  user-select: none;
}

.vue-grid-item:hover {
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}

.vue-grid-item--dragging {
  z-index: 1000;
  opacity: 0.8;
}

.vue-grid-item--resizing {
  z-index: 999;
}

.vue-grid-item-content {
  height: 100%;
  overflow: hidden;
}

.vue-grid-placeholder {
  background: rgba(64, 158, 255, 0.3) !important;
  border: 2px dashed #409eff !important;
  border-radius: 6px;
  z-index: 2;
}

.default-widget {
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
  flex: 1;
}

.resize-handle {
  background: transparent;
  border: none;
  color: white;
  font-size: 14px;
  cursor: se-resize;
  padding: 2px 6px;
  border-radius: 3px;
  transition: background-color 0.2s;
}

.resize-handle:hover {
  background: rgba(255, 255, 255, 0.2);
}

.widget-body {
  flex: 1;
  padding: 12px;
  font-size: 12px;
  color: #666;
  overflow: auto;
}

@media (max-width: 768px) {
  .vue-grid-layout {
    min-height: 300px;
  }
}
</style>
