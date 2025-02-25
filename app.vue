<template>
  <div>
    <NuxtRouteAnnouncer />
    <NuxtPage />
  </div>
</template>
<script lang="ts" setup>

useHead({
  link: [
    {
      rel: 'stylesheet',
      href: 'https://cdn.jsdelivr.net/npm/@picocss/pico@2/css/pico.min.css',
    },
  ],
})
</script>

<style>
</style>