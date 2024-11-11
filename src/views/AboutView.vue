<script setup>
import Player from '@/components/player-components/QNAComponent.vue'
</script>

<template>
  <div class="about">
    <Player></Player>
  </div>
</template>

<style>
@media (min-width: 1024px) {
  .about {
    min-height: 100vh;
    display: flex;
    align-items: center;
  }
}
</style>
