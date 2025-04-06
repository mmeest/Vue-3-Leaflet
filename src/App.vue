<script setup lang="ts">
import Map from './components/Map.vue'
</script>

<template>
  <div class="app-container">
    <Map />
  </div>
</template>

<style>
html, body {
  margin: 0;
  padding: 0;
  width: 100%;
  height: 100%;
  overflow: hidden;
}

.app-container {
  width: 100vw;
  height: 100vh;
  position: relative;
  overflow: hidden;
}
</style>