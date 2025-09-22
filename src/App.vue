<template>
  <div class="app">
    <TitleGenerator />
  </div>
</template>

<script lang="ts" setup>
import TitleGenerator from './components/TitleGenerator.vue'
</script>
<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  background-color: #fafafa;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
}

.app {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  background: linear-gradient(135deg, #fdf6f0 0%, #f9f2f4 100%);
}
</style>
