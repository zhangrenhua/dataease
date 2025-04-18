<template>
  <el-col style="padding: 0 5px 0 5px;">
    <el-row>
      <span class="header-title">{{ $t('panel.share_in') }}</span>
      <div class="block" style="margin-top:8px;">
        <el-tree :data="datas" :props="defaultProps" :highlight-current="true" node-key="name" :default-expanded-keys="expandNodes" @node-click="handleNodeClick">
          <span slot-scope="{ data }" class="custom-tree-node father">
            <span style="display: flex; flex: 1 1 0%; width: 0px;" :class="!!data.msgNode ? 'msg-node-class': ''">
              <span v-if="!!data.id">
                <svg-icon icon-class="panel" class="ds-icon-scene" />
              </span>
              <span style="margin-left: 6px;white-space:nowrap;overflow:hidden;text-overflow:ellipsis;">{{ data.name }}</span>
            </span>
            <!--  <span :class="!!data.msgNode ? 'msg-node-class': ''">
              <span v-if="!!data.id">
                <el-button
                  icon="el-icon-picture-outline"
                  type="text"
                />
              </span>
              <span style="margin-left: 6px">{{ data.name }}</span>
            </span> -->
          </span>
        </el-tree>
      </div>
    </el-row>

    <el-row>
      <span class="header-title">{{ $t('panel.share_out') }}</span>
      <div class="block" style="margin-top:8px;">
        <el-tree :data="outDatas" :props="defaultProps" :highlight-current="true" node-key="name" :default-expand-all="true" @node-click="viewMyShare">
          <span slot-scope="{ data }" class="custom-tree-node father">
            <span style="display: flex; flex: 1 1 0%; width: 0px;">
              <span v-if="!!data.id">
                <svg-icon icon-class="panel" class="ds-icon-scene" />
              </span>
              <span style="margin-left: 6px;white-space:nowrap;overflow:hidden;text-overflow:ellipsis;">{{ data.name }}</span>
            </span>

            <span class="child">
              <span class="el-dropdown-link">
                <el-button
                  icon="el-icon-delete"
                  type="text"
                  size="small"
                  @click="removeCurrent(data)"
                />
              </span>
            </span>
          </span>

        </el-tree>
      </div>
    </el-row>
  </el-col>
</template>

<script>
import { loadTree, loadShareOutTree, removeShares } from '@/api/panel/share'
import { uuid } from 'vue-uuid'
import { get } from '@/api/panel/panel'
import bus from '@/utils/bus'
export default {
  name: 'ShareTree',
  props: {
    msgPanelIds: {
      type: Array,
      default: null
    }
  },
  data() {
    return {
      datas: [],
      defaultProps: {
        children: 'children',
        label: 'name'
      },
      expandNodes: [],
      outDatas: []
    }
  },
  created() {
    this.initData().then(res => {
      this.datas = res.data
      if (this.msgPanelIds && this.msgPanelIds.length > 0) {
        this.expandMsgNode(this.msgPanelIds)
      }
    })
    this.initOutData().then(res => {
      this.outDatas = res.data
    })
  },

  methods: {
    initData() {
      const param = {}
      return loadTree(param)
    },
    initOutData() {
      return loadShareOutTree()
    },
    handleNodeClick(data) {
      get('panel/group/findOne/' + data.id).then(response => {
        this.$store.commit('setComponentData', this.resetID(JSON.parse(response.data.panelData)))
        this.$store.commit('setCanvasStyle', JSON.parse(response.data.panelStyle))

        this.$store.dispatch('panel/setPanelInfo', data)
        bus.$emit('set-panel-show-type', 1)
      })
    },
    viewMyShare(data) {
      get('panel/group/findOne/' + data.id).then(response => {
        this.$store.commit('setComponentData', this.resetID(JSON.parse(response.data.panelData)))
        this.$store.commit('setCanvasStyle', JSON.parse(response.data.panelStyle))

        this.$store.dispatch('panel/setPanelInfo', data)
        bus.$emit('set-panel-show-type', 2)
      })
    },
    resetID(data) {
      if (data) {
        data.forEach(item => {
          item.type !== 'custom' && (item.id = uuid.v1())
        })
      }

      return data
    },
    expandMsgNode(panelIds) {
      // console.log(panelIds)
      this.$nextTick(() => {
        this.getMsgNodes(panelIds)
      })
    },
    getMsgNodes(panelIds) {
      this.datas.forEach(item => {
        if (item.children && item.children.length > 0) {
          item.children.forEach(node => {
            if (panelIds.includes(node.id)) {
              node.msgNode = true
              this.expandNodes.push(item.name)
            }
          })
        }
      })
    },
    removeCurrent(node) {
      console.log(node)
      const param = {
        panelId: node.id
      }

      this.$confirm(this.$t('panel.remove_share_confirm'), '', {
        confirmButtonText: this.$t('commons.confirm'),
        cancelButtonText: this.$t('commons.cancel'),
        type: 'warning'
      }).then(() => {
        removeShares(param).then(res => {
          this.initOutData().then(res => {
            this.outDatas = res.data
          })
          this.$success(this.$t('commons.delete_success'))
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: this.$t('commons.delete_cancelled')
        })
      })
    }

  }
}
</script>

<style lang="scss" scoped>
.header-title {
    font-size: 14px;
    flex: 1;
    color: var(--TextPrimary, #606266);
    font-weight: bold;
    display: block;
    height: 100%;
    /*line-height: 36px;*/
  }
.msg-node-class {
  color: red;
  >>> i{
    color: red;
  }
}
 .custom-tree-node {
    flex: 1;
    display: flex;
    align-items: center;
    justify-content: space-between;
    font-size: 14px;
    padding-right:8px;
  }

  .custom-tree-node-list {
    flex: 1;
    display: flex;
    align-items: center;
    justify-content: space-between;
    font-size: 14px;
    padding:0 8px;
  }
  .father .child {
    /*display: none;*/
    visibility: hidden;
  }
  .father:hover .child {
    /*display: inline;*/
    visibility: visible;
  }
</style>
