<script setup lang="ts">
import { RouterLink, RouterView } from 'vue-router'
</script>

<template>
  <div style="display: flex;">
    <div style="margin-right: 10px;">
      <RouterLink to="/">Home</RouterLink>
    </div>
    <div>
      <RouterLink to="/about">About</RouterLink>
    </div>

  </div>

  <RouterView />
</template>

