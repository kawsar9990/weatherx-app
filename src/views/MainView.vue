<template>
  <div class="block md:hidden">
    <MobileView />
  </div>
  <div class="hidden md:block">
    <DesktopView />
  </div>
</template>

<script setup lang="ts">
import MobileView from './MobileView.vue';
import DesktopView from './DesktopView.vue';
</script>