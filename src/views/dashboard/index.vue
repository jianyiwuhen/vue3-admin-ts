<template>
  <div class="scroll-y">
    <div class="dashboard-container">
      <div class="dashboard-text">name: {{ username }}</div>
    </div>
    <div>update element-plus to 1.2.+, add svg icon, icon-font is abandoned</div>
    <i class="el-icon-add-location">abandoned</i>
    <div>
      <el-icon>
        <Svg.Edit></Svg.Edit>
      </el-icon>
    </div>
  </div>
</template>

<script setup lang="ts">
import * as Svg from '@element-plus/icons'
import { computed } from 'vue'
import { useStore } from 'vuex'

let store = useStore()
const username = computed(() => {
  return store.state.user.username
})
</script>

<style lang="scss" scoped>
.dashboard {
  &-text {
    font-size: 30px;
    line-height: 46px;
  }
}
</style>
