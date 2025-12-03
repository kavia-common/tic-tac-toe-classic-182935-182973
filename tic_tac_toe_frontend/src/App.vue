<script setup lang="ts">
// Root shell only renders the router view to keep a clean SPA.
// Theme colors are provided via CSS variables in assets and component styles.
import { RouterView } from 'vue-router'
</script>

<template>
  <RouterView />
</template>

<style scoped>
/* Keep root minimal to let views control page layout */
</style>
