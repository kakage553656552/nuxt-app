<template>
  <div>
    <publicPdf
      :pdf-data="pdfData"
      :col-num="12"
      :row-height="10"
      :is-draggable="true"
      :is-resizable="true"
      :vertical-compact="true"
      :margin="[10, 10]"
      :use-css-transforms="true"
      @layout-updated="onLayoutUpdated"
    >
      <template #default="{ item }">
        <div class="custom-widget">
          <!-- <div class="widget-header">
            <h4>{{ item.title }}</h4>
            <button class="remove-btn" @click="removeWidget(item.i)">
              ×
            </button>
          </div> -->
          <!-- <div class="">
            <p>{{ item.content }}</p>
          </div> -->
          {{item.name}}222
        </div>
      </template>
    </publicPdf>

    <!-- <div class="control-panel">
      <el-button type="primary" @click="addWidget">
        添加Widget
      </el-button>
      <el-button type="success" @click="saveLayout">
        保存布局
      </el-button>
      <el-button type="info" @click="resetLayout">
        重置布局
      </el-button>
    </div> -->
  </div>
</template>

<script>
export default {
  name: 'IndexPage',
  data () {
    return {
      pdfData: [
        {
          data: [
            {
              h: 3,
              i: '40daef5e-130d-460f-abd4-3a8158c0f6ba',
              w: 12,
              x: 0,
              y: 0,
              id: '1338421359567159298',
              com: 'pageheadergadget',
              moved: false,
              static: true,
              slicers: []
            },
            {
              h: 4,
              i: 'c8bf797f-7649-4063-80cf-24c22be79846',
              w: 4,
              x: 0,
              y: 3,
              id: '1352302658778537986',
              com: 'cardnumbergadget',
              name: 'Open Tickets',
              moved: false,
              isCompany: false,
              allowPaging: null,
              businessType: 'Template',
              integrationList: [
                {
                  name: 'ConnectWise Manage - API',
                  code: 'CW',
                  icon: 'https://oss.mspbots.ai/files/integration_logo/connectwise_manage_logo.png'
                }
              ],
              isHidden: 0,
              integrationCode: ['CW'],
              slicers: []
            },
            {
              h: 4,
              i: 'card-2',
              w: 4,
              x: 4,
              y: 3,
              com: 'cardnumbergadget',
              name: 'Closed Tickets',
              businessType: 'Standard',
              integrationList: [
                {
                  name: 'ConnectWise Manage - API',
                  code: 'CW',
                  icon: 'https://oss.mspbots.ai/files/integration_logo/connectwise_manage_logo.png'
                }
              ]
            },
            {
              h: 4,
              i: 'card-3',
              w: 4,
              x: 8,
              y: 3,
              com: 'cardnumbergadget',
              name: 'Revenue',
              businessType: 'Premium',
              integrationList: [
                {
                  name: 'QuickBooks - API',
                  code: 'QB',
                  icon: 'https://oss.mspbots.ai/files/integration_logo/quickbooks_logo.png'
                }
              ]
            }
          ],
          slicers: []
        },
        {
          data: [
            {
              h: 6,
              i: 'chart-1',
              w: 8,
              x: 0,
              y: 0,
              com: 'cardnumbergadget',
              name: 'Monthly Revenue Trend',
              businessType: 'Analytics'
            },
            {
              h: 3,
              i: 'card-4',
              w: 4,
              x: 8,
              y: 0,
              com: 'cardnumbergadget',
              name: 'Active Projects',
              businessType: 'Standard'
            },
            {
              h: 3,
              i: 'card-5',
              w: 4,
              x: 8,
              y: 3,
              com: 'cardnumbergadget',
              name: 'Team Members',
              businessType: 'Standard'
            }
          ],
          slicers: []
        }
      ]
    }
  },
  mounted () {
  },
  methods: {
    onLayoutUpdated (newLayout) {
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
  width: 100%;
  height: 100%;
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
