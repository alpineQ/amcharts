<template>
  <div style="height: 100vh; width: 100wv; background-color: #191b1c;">
    <WorldMap code="US" :width="500" :height="500" />
  </div>
</template>

<script setup lang="ts">
import WorldMap from './components/WorldMap.vue'
</script>