<template>
  <div class="pdf-viewer-container">
    <div class="pdf-document">
      <!-- PDF页面渲染 - 连续滚动布局 -->
      <div
        v-for="(page, pageIndex) in pdfPages"
        :id="`page-${pageIndex}`"
        :key="pageIndex"
        class="pdf-page"
      >
        <!-- 页面头部只在第一页显示 -->
        <div v-if="pageIndex === 0 && headerData" class="page-header">
          <PageHeaderGadget :item="headerData" />
        </div>

        <!-- 页面内容区域 -->
        <div class="page-content">
          <div v-if="page.length > 0" class="vue-grid-layout">
            <div
              v-for="item in page"
              :key="item.i"
              :style="getItemStyle(item)"
              class="vue-grid-item"
              :class="{
                'vue-grid-item--dragging': isDragging && draggedItem && draggedItem.i === item.i,
                'vue-grid-item--resizing': isResizing && resizingItem && resizingItem.i === item.i,
                'vue-grid-item--static': item.static
              }"
              :draggable="isDraggable && !item.static"
              @dragstart="onDragStart($event, item)"
              @dragend="onDragEnd"
            >
              <div class="vue-grid-item-content">
                <slot
                  :item="item"
                  :index="getItemIndex(item)"
                >
                  <component
                    :is="getComponentName(item.com)"
                    :item="item"
                  />
                </slot>
              </div>
            </div>

            <div
              v-if="dragPlaceholder.visible"
              :style="dragPlaceholder.style"
              class="vue-grid-placeholder"
            />
          </div>
        </div>

        <!-- 页脚 -->
        <!-- <div class="page-footer">
          <div class="page-info">
            <span class="page-number">{{ pageIndex + 1 }}</span>
            <span class="total-pages">/ {{ pdfPages.length }}</span>
          </div>
        </div> -->
      </div>
    </div>

    <!-- 侧边页面缩略图导航 -->
    <div v-if="pdfPages.length > 1" class="pdf-sidebar">
      <div class="sidebar-header">
        <h4>页面</h4>
      </div>
      <div class="page-thumbnails">
        <div
          v-for="(page, index) in pdfPages"
          :key="index"
          class="page-thumbnail"
          :class="{ 'active': isPageInView(index) }"
          @click="scrollToPage(index)"
        >
          <div class="thumbnail-preview">
            <span class="thumbnail-number">{{ index + 1 }}</span>
          </div>
          <span class="thumbnail-label">第{{ index + 1 }}页</span>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'PublicPdf',
  components: {
    PageHeaderGadget: () => import('./PageHeaderGadget.vue'),
    CardNumberGadget: () => import('./CardNumberGadget.vue')
  },
  props: {
    layout: {
      type: Array,
      required: true
    },
    pdfData: {
      type: Array,
      default: () => []
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
      },
      currentPage: 0,
      visiblePages: []
    }
  },
  computed: {
    colWidth () {
      // A4页面宽度为210mm，减去页边距
      const pageWidth = 210 - 40 // 40mm页边距
      return (pageWidth - (this.colNum + 1) * this.margin[0]) / this.colNum
    },
    currentBreakpoint () {
      if (!this.responsive) {
        return 'lg'
      }
      const width = this.containerWidth
      const breakpoints = this.breakpoints
      return Object.keys(breakpoints)
        .sort((a, b) => breakpoints[b] - breakpoints[a])
        .find(bp => width >= breakpoints[bp]) || 'xxs'
    },
    currentCols () {
      return this.responsive ? this.cols[this.currentBreakpoint] : this.colNum
    },
    headerData () {
      if (this.pdfData.length > 0) {
        const firstItem = this.pdfData[0]
        if (firstItem.com === 'pageheadergadget') {
          return firstItem
        }
      }
      return null
    },
    pdfPages () {
      if (this.pdfData.length === 0) {
        return [this.layout]
      }

      const pages = []
      const headerIndex = this.pdfData.findIndex(item => item.com === 'pageheadergadget')

      // 跳过header，处理其他页面数据
      for (let i = headerIndex + 1; i < this.pdfData.length; i++) {
        const pageData = this.pdfData[i]
        if (pageData.data && Array.isArray(pageData.data)) {
          pages.push(pageData.data)
        } else if (pageData.i) {
          // 单个item
          if (pages.length === 0) { pages.push([]) }
          pages[pages.length - 1].push(pageData)
        }
      }

      return pages.length > 0 ? pages : [this.layout]
    }
  },
  mounted () {
    this.updateContainerWidth()
    window.addEventListener('resize', this.updateContainerWidth)
    document.addEventListener('mousemove', this.onMouseMove)
    document.addEventListener('mouseup', this.onMouseUp)
    this.setupScrollObserver()
  },
  beforeDestroy () {
    window.removeEventListener('resize', this.updateContainerWidth)
    document.removeEventListener('mousemove', this.onMouseMove)
    document.removeEventListener('mouseup', this.onMouseUp)
  },
  methods: {
    updateContainerWidth () {
      if (this.$el) {
        // A4页面宽度210mm，转换为像素（约等于794px在96dpi下）
        this.containerWidth = 794
      }
    },

    getItemStyle (item) {
      const colWidth = this.colWidth
      const rowHeight = this.rowHeight
      const margin = this.margin

      // 转换mm为像素（1mm ≈ 3.78px 在96dpi下）
      const mmToPx = 3.78
      const x = item.x * colWidth * mmToPx + (item.x + 1) * margin[0]
      const y = item.y * rowHeight + (item.y + 1) * margin[1]
      const width = item.w * colWidth * mmToPx + (item.w - 1) * margin[0]
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
      const mmToPx = 3.78

      const newX = Math.max(0, Math.round((event.clientX - containerRect.left - this.dragOffset.x) / ((colWidth * mmToPx) + margin[0])))
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
      const mmToPx = 3.78

      const newW = Math.max(1, this.resizeStartSize.w + Math.round(deltaX / ((colWidth * mmToPx) + margin[0])))
      const newH = Math.max(1, this.resizeStartSize.h + Math.round(deltaY / (rowHeight + margin[1])))

      this.resizingItem.w = Math.min(newW, this.currentCols - this.resizingItem.x)
      this.resizingItem.h = newH
    },

    updatePlaceholder (item) {
      this.dragPlaceholder.style = this.getItemStyle(item)
    },

    getComponentName (componentType) {
      const componentMap = {
        pageheadergadget: 'PageHeaderGadget',
        cardnumbergadget: 'CardNumberGadget'
      }
      return componentMap[componentType] || 'div'
    },

    getItemIndex (item) {
      const currentPageData = this.pdfPages[this.currentPage]
      return currentPageData.indexOf(item)
    },

    // 滚动相关方法
    setupScrollObserver () {
      const observer = new IntersectionObserver((entries) => {
        this.visiblePages = entries
          .filter(entry => entry.isIntersecting)
          .map(entry => parseInt(entry.target.id.replace('page-', '')))
      }, {
        threshold: 0.5
      })

      this.$nextTick(() => {
        const pages = document.querySelectorAll('.pdf-page')
        pages.forEach(page => observer.observe(page))
      })
    },

    scrollToPage (pageIndex) {
      const pageElement = document.getElementById(`page-${pageIndex}`)
      if (pageElement) {
        pageElement.scrollIntoView({
          behavior: 'smooth',
          block: 'start'
        })
      }
    },

    isPageInView (pageIndex) {
      return this.visiblePages.includes(pageIndex)
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

<style>
html, body {
  margin: 0;
  padding: 0;
}
</style>

<style scoped>
.pdf-viewer-container {
  display: flex;
  height: 100vh;
  background: #f5f5f5;
}

.pdf-document {
  flex: 1;
  overflow-y: auto;
  overflow-x: hidden;
  padding: 20px;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.pdf-page {
  width: 210mm;
  min-height: 297mm;
  background: white;
  margin-bottom: 20mm;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.15);
  position: relative;
  padding: 20mm;
  box-sizing: border-box;
}

.page-header {
  margin-bottom: 15mm;
}

.page-content {
  min-height: 220mm;
  position: relative;
}

.vue-grid-layout {
  position: relative;
  width: 100%;
  min-height: 200mm;
}

.page-footer {
  position: absolute;
  bottom: 10mm;
  left: 20mm;
  right: 20mm;
  text-align: center;
  border-top: 1px solid #e5e7eb;
  padding-top: 5mm;
}

.page-info {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 2px;
  font-size: 10pt;
  color: #6b7280;
}

.page-number {
  font-weight: 600;
}

.total-pages {
  font-weight: 400;
}

/* 侧边栏样式 */
.pdf-sidebar {
  width: 200px;
  background: white;
  border-left: 1px solid #e5e7eb;
  overflow-y: auto;
  flex-shrink: 0;
}

.sidebar-header {
  padding: 16px;
  border-bottom: 1px solid #e5e7eb;
  background: #f9fafb;
}

.sidebar-header h4 {
  margin: 0;
  font-size: 14px;
  font-weight: 600;
  color: #374151;
}

.page-thumbnails {
  padding: 16px 12px;
}

.page-thumbnail {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 12px 8px;
  margin-bottom: 12px;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.2s;
  border: 2px solid transparent;
}

.page-thumbnail:hover {
  background: #f3f4f6;
}

.page-thumbnail.active {
  background: #eff6ff;
  border-color: #3b82f6;
}

.thumbnail-preview {
  width: 40px;
  height: 56px;
  background: white;
  border: 1px solid #d1d5db;
  border-radius: 4px;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 8px;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
}

.thumbnail-number {
  font-size: 12px;
  font-weight: 600;
  color: #6b7280;
}

.thumbnail-label {
  font-size: 11px;
  color: #6b7280;
  text-align: center;
}

.page-thumbnail.active .thumbnail-number,
.page-thumbnail.active .thumbnail-label {
  color: #3b82f6;
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

.vue-grid-item--static {
  cursor: default !important;
}

.vue-grid-item--static:hover {
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1) !important;
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

/* 打印样式 */
@media print {
  * {
    -webkit-print-color-adjust: exact !important;
    print-color-adjust: exact !important;
  }

  .pdf-viewer-container {
    height: auto;
  }

  .pdf-sidebar {
    display: none;
  }

  .pdf-document {
    padding: 0;
    display: block;
  }

  .pdf-page {
    margin-bottom: 0;
    box-shadow: none;
    page-break-after: auto;
    page-break-inside: avoid;
    height: auto;
    min-height: auto;
    max-height: none;
    overflow: visible;
  }

  .pdf-page:last-child {
    page-break-after: avoid;
  }

  .page-content {
    min-height: auto;
  }

  .vue-grid-layout {
    min-height: auto;
  }

  .page-footer {
    position: relative;
    margin-top: 20mm;
  }
}

/* 响应式设计 */
@media (max-width: 1200px) {
  .pdf-page {
    width: 90vw;
    min-height: calc(90vw * 1.414);
    padding: 5vw;
  }
}

@media (max-width: 768px) {
  .pdf-sidebar {
    display: none;
  }

  .pdf-page {
    width: 95vw;
    min-height: calc(95vw * 1.414);
    padding: 4vw;
  }
}
</style>
