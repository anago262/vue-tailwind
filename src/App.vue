<script setup>
import { ref } from "vue";
import SliderBar from "./components/SliderBar.vue";
import SimpleBar from "./components/SimpleBar.vue";

const score = ref(50);
</script>

<template>
  <div class="flex flex-col items-center mt-8">
    <SliderBar v-model="score" barHeight="20px" dotSize="24px" knobColor="#f87171" />
    <div class="h-8"></div>
    <!-- knobSizeを指定してつまみの大きさを変える例 -->
    <SliderBar v-model="score" barHeight="20px" dotSize="24px" knobColor="#34d399" :knobSize="40" />
    <div class="h-8"></div>
    <SliderBar v-model="score" barHeight="20px" dotSize="24px" knobColor="#fbbf24" :knobSize="20" />
    <div class="h-8"></div>
    <!-- dotSizeを指定して目盛りの大きさを変える例 -->
    <SliderBar v-model="score" barHeight="20px" dotSize="36px" knobColor="#6366f1" :knobSize="44" />
    <div class="h-8"></div>
    <SliderBar v-model="score" barHeight="20px" dotSize="16px" knobColor="#10b981" :knobSize="24" />

    <br/>
    <br/>

    <SimpleBar v-model="score" barHeight="20px" dotSize="24px" knobColor="#f87171"/>
  </div>
</template>
