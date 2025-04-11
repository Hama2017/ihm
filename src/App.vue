// src/App.vue
<template>
  <div class="app min-h-screen bg-gray-50">
    <ContractEditor />
  </div>
</template>

<script setup lang="ts">
import ContractEditor from './components/ContractEditor.vue';
</script>

<style>
/* import the necessary styles for Vue Flow to work */
@import '@vue-flow/core/dist/style.css';

/* import the default theme, this is optional but generally recommended */
@import '@vue-flow/core/dist/theme-default.css';

body {
  margin: 0;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
}
</style>