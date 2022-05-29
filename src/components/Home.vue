<template>
  <div>
    <el-button type="info" @click='logout'>退出</el-button>
  </div>
</template>

<script>
export default {
  methods: {
    logout() {
      // 清空缓存的 token
      sessionStorage.clear()
      this.$router.push('/login')
    }
  }
}
</script>

<style lang="less" scoped>

</style>