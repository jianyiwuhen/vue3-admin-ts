<template>
  <div>
    <div>ref1:{{ ref1 }},ref2:{{ ref2 }}</div>
    <el-button @click="hookExample">执行hook方法</el-button>
  </div>
</template>

<script setup lang="ts">
import testHooks from '@/hooks/testHooks'
let { ref1, ref2, hooksFunc } = testHooks()
const hookExample = () => {
  hooksFunc()
}
console.log(`获取到hook导出的数据${ref2}`)
console.log(`获取到hook导出的数据${ref1}`)
</script>

<style scoped lang="scss"></style>
