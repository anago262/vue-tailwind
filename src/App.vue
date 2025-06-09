<script setup>
import { ref } from "vue";
import SliderBar from "./components/SliderBar.vue";

const score = ref(50);
</script>

<template>
  <div class="flex flex-col items-center mt-8">
    <SliderBar v-model="score" barHeight="20px" dotSize="24px" knobColor="#f87171" />
    <div class="mt-4 text-gray-600">現在のスコア: {{ score }}</div>
  </div>
</template>
