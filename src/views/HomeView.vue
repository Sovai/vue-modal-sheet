<script setup>
import TheSheet from "@/components/TheSheet.vue";
</script>

<template>
  <TheSheet title="sheet title">
    <div v-for="i in 60" :key="i">
      Lorem ipsum, dolor sit amet consectetur adipisicing elit. Aliquid, rerum
      voluptates illum ratione porro quidem. Aliquid laudantium magnam placeat
      recusandae unde, velit nobis error eius odio minima sed! Porro, fugiat.
    </div>
  </TheSheet>
</template>
