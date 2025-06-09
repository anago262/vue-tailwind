<script setup>
import { ref } from "vue";
import SliderBar from "./components/SliderBar.vue";
import SimpleBar from "./components/SimpleBar.vue";

const score = ref(50);
</script>

<template>
  <div class="flex flex-col items-center mt-8">
    <SliderBar v-model="score" barHeight="20px" dotSize="24px" knobColor="#f87171" />
    <br/>    <br/>
    <br/>
    <br/>

    <SimpleBar v-model="score" barHeight="20px" dotSize="24px" knobColor="#f87171"/>
  </div>
</template>
