<template>
  <div class="h-screen">
    <TopBar />
    <Nuxt />
  </div>
</template>

<script>
import TopBar from "~/components/TopBar.vue";
export default {
  components: { TopBar },
};
</script>

<style>
</style>