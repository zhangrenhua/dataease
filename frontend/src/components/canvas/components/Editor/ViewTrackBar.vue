<template>
  <div>
    <el-dropdown trigger="click">
      <input id="input" ref="trackButton" type="button" hidden>
      <el-dropdown-menu class="track-menu" :append-to-body="false">
        <el-dropdown-item v-for="(item, key) in trackMenu" :key="key" @click.native="trackMenuClick(item)"><span class="menu-item">{{ i18n_map[item] }}</span></el-dropdown-item>
      </el-dropdown-menu>
    </el-dropdown>
  </div>
</template>

<script>

export default {
  props: {
    trackMenu: {
      type: Array,
      required: true
    }
  },
  data() {
    return {
      i18n_map: {
        drill: this.$t('panel.drill'),
        linkage: this.$t('panel.linkage'),
        jump: this.$t('panel.jump')
      }
    }
  },
  computed: {
  },
  methods: {
    trackButtonClick() {
      this.$refs.trackButton.click()
    },
    trackMenuClick(menu) {
      this.$emit('trackClick', menu)
    }
  }
}
</script>

<style lang="scss" scoped>
  .menu-item {
    font-size: 12px;
  }

  ::v-deep ul {
    width: 80px;
  }

  .track-menu {
    border: #3a8ee6 1px solid;
  }

</style>
