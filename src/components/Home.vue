<template>
 <!-- <div>
    <el-button type="info" @click="logout">退出</el-button>
  </div> -->
  <el-container>
    <!-- 头部区域 -->
    <el-header>Header</el-header>
    <!-- 页面主体区域 -->
    <el-container>
      <!-- 侧边栏 -->
      <el-aside width="200px">Aside</el-aside>
      <!-- 右边主体区域 -->
      <el-main>Main</el-main>
    </el-container>
  </el-container>
</template>

<script>
export default {
  methods: {
    logout () {
      window.sessionStorage.clear()
      this.$router.push('/login')
    }
  }
}
</script>

<style>
</style>
