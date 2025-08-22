<template>
  <div id="app">
    <DailyInspiration />
  </div>
</template>

<script setup>
import DailyInspiration from './components/DailyInspiration.vue'
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  background-color: #0f0f0f;
  color: #e4e4e7;
  line-height: 1.6;
}

#app {
  min-height: 100vh;
}
</style>
