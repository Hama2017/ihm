// src/components/contract/StateNode.vue
<template>
  <div class="state-node bg-indigo-700 text-white p-4 rounded-md w-40 flex flex-col items-center">
    <div class="connector top w-3 h-3 bg-white rounded-full absolute top-0 left-1/2 transform -translate-x-1/2 -translate-y-1/2"></div>
    {{ label }}
    <div class="connector bottom w-3 h-3 bg-white rounded-full absolute bottom-0 left-1/2 transform -translate-x-1/2 translate-y-1/2"></div>
  </div>
</template>

<script setup lang="ts">
defineProps<{
  label: string;
}>();
</script>

<style scoped>
.state-node {
  position: relative;
  text-align: center;
  user-select: none;
  cursor: move;
}

.connector {
  cursor: crosshair;
}
</style>