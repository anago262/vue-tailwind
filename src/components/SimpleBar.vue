<template>
  <div class="w-full flex flex-col items-center mt-16">
    <div class="flex items-center w-full justify-center space-x-8 mb-4">
      <!-- 左側：スコアinput -->
      <div class="flex flex-col items-center">
        <span class="mb-1 text-lg text-gray-600">スコア</span>
        <input
          type="number"
          class="text-2xl font-bold text-blue-700 border-b-2 border-blue-300 focus:outline-none w-24 text-center"
          :min="dotValues[1]"
          :max="dotValues[dotValues.length - 2]"
          :step="scoreStep"
          v-model="inputValue"
          @change="onInputChange"
        />
      </div>
      <!-- 中央：バー本体 -->
      <div ref="barRef" class="relative w-[500px] h-2 bg-gray-300 rounded-full flex items-center">
        <!-- 目盛り -->
        <div
          v-for="i in dotCount"
          :key="i"
          class="absolute bg-white border-2 border-gray-400 rounded-full cursor-pointer transform -translate-x-1/2 -translate-y-1/2"
          :style="{
            width: dotSizePx,
            height: dotSizePx,
            left: `${((i - 1) / (dotCount - 1)) * 100}%`,
            top: '50%'
          }"
          @click.stop="moveKnob(i - 1)"
        ></div>
        <!-- スコアラベル -->
        <div
          v-for="i in dotCount"
          :key="`label-${i}`"
          class="absolute text-sm text-gray-700 transform -translate-x-1/2"
          :style="{
            left: `${((i - 1) / (dotCount - 1)) * 100}%`,
            top: 'calc(50% + 16px)'
          }"
        >
          {{ (i - 1) * scoreStep }}
        </div>
        <!-- つまみ -->
        <div
          class="absolute transition-all duration-200 transform -translate-x-1/2 -translate-y-1/2"
          :style="{
            width: knobSizePx,
            height: knobSizePx,
            background: knobColor,
            border: `3px solid ${knobColor}`,
            borderRadius: '50%',
            cursor: 'pointer',
            left: `${(knobIndex / (dotCount - 1)) * 100}%`,
            top: '50%'
          }"
          @mousedown.prevent="startDrag"
          @touchstart.prevent="startDrag"
        ></div>
      </div>
      <!-- 右側：スコア2input -->
      <div class="flex flex-col items-center">
        <span class="mb-1 text-lg text-gray-600">スコア2</span>
        <input
          type="number"
          class="text-2xl font-bold text-pink-700 border-b-2 border-pink-300 focus:outline-none w-24 text-center"
          :min="1"
          :max="99"
          :step="scoreStep"
          v-model="inputValue2"
          @change="onInput2Change"
        />
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onBeforeUnmount, watch } from 'vue';

const props = defineProps({
  dotCount: { type: Number, default: 11 },
  scoreStep: { type: Number, default: 10 },
  dotSize: { type: [Number, String], default: 24 },
  knobSize: { type: [Number, String], default: null },
  knobColor: { type: String, default: '#0ea5e9' }
});

const dotCount = props.dotCount;
const scoreStep = props.scoreStep;
const dotValues = Array.from({ length: dotCount }, (_, i) => i * scoreStep);
const minIndex = 1;
const maxIndex = dotCount - 2;
const knobIndex = ref(minIndex);
const barRef = ref(null);
const dragging = ref(false);

const dotSizePx = computed(() =>
  typeof props.dotSize === 'number' ? `${props.dotSize}px` : props.dotSize
);

const knobSizePx = computed(() => {
  if (props.knobSize !== null && props.knobSize !== undefined) {
    return typeof props.knobSize === 'number'
      ? `${props.knobSize}px`
      : props.knobSize;
  }
  return `calc(${dotSizePx.value} + 8px)`;
});

// 現在のスコア
const currentScore = computed(() => dotValues[knobIndex.value]);
// スコア2
const score2 = computed(() => 100 - currentScore.value);

const inputValue = ref(currentScore.value);
const inputValue2 = ref(score2.value);

// knobIndexが変わったらinputValueも同期
watch(knobIndex, () => {
  inputValue.value = currentScore.value;
  inputValue2.value = 100 - currentScore.value;
});

// inputValueが変わったらknobIndexも同期
function onInputChange() {
  let val = Number(inputValue.value);
  if (isNaN(val)) val = dotValues[minIndex];
  val = Math.max(dotValues[minIndex], Math.min(val, dotValues[maxIndex]));
  // 10刻みにスナップ
  let idx = dotValues.findIndex(v => v === val);
  if (idx === -1) {
    let minDiff = Infinity;
    dotValues.forEach((v, i) => {
      const diff = Math.abs(v - val);
      if (diff < minDiff) {
        minDiff = diff;
        idx = i;
      }
    });
  }
  // 端は選択不可
  if (idx <= 0) idx = 1;
  if (idx >= dotValues.length - 1) idx = dotValues.length - 2;
  knobIndex.value = idx;
  inputValue.value = dotValues[idx];
  inputValue2.value = 100 - dotValues[idx];
}

// スコア2入力欄の変更
function onInput2Change() {
  let val2 = Number(inputValue2.value);
  if (isNaN(val2)) val2 = 1;
  val2 = Math.max(1, Math.min(val2, 99));
  // スコア = 100 - スコア2
  let score = 100 - val2;
  // 10刻みにスナップ
  let idx = dotValues.findIndex(v => v === score);
  if (idx === -1) {
    let minDiff = Infinity;
    dotValues.forEach((v, i) => {
      const diff = Math.abs(v - score);
      if (diff < minDiff) {
        minDiff = diff;
        idx = i;
      }
    });
  }
  // 端は選択不可
  if (idx <= 0) idx = 1;
  if (idx >= dotValues.length - 1) idx = dotValues.length - 2;
  knobIndex.value = idx;
  inputValue.value = dotValues[idx];
  inputValue2.value = 100 - dotValues[idx];
}

function moveKnob(idx) {
  // 端は選択不可
  if (idx <= 0 || idx >= dotValues.length - 1) return;
  knobIndex.value = idx;
}

function onBarClick(event) {
  updateIndexByClientX(
    event.clientX,
    barRef.value.getBoundingClientRect().left,
    barRef.value.clientWidth
  );
}

function startDrag(event) {
  dragging.value = true;
  const moveEvent = event.type === 'mousedown' ? 'mousemove' : 'touchmove';
  const upEvent = event.type === 'mousedown' ? 'mouseup' : 'touchend';
  window.addEventListener(moveEvent, onDrag);
  window.addEventListener(upEvent, stopDrag);
}

function onDrag(event) {
  if (!dragging.value) return;
  const clientX = event.touches ? event.touches[0].clientX : event.clientX;
  updateIndexByClientX(
    clientX,
    barRef.value.getBoundingClientRect().left,
    barRef.value.clientWidth
  );
}

function stopDrag() {
  dragging.value = false;
  window.removeEventListener('mousemove', onDrag);
  window.removeEventListener('mouseup', stopDrag);
  window.removeEventListener('touchmove', onDrag);
  window.removeEventListener('touchend', stopDrag);
}

function updateIndexByClientX(clientX, barLeft, barWidth) {
  let x = clientX - barLeft;
  x = Math.max(0, Math.min(x, barWidth));
  const ratio = x / barWidth;
  let idx = Math.round(ratio * (dotCount - 1));
  // 端は選択不可
  idx = Math.min(Math.max(idx, minIndex), maxIndex);
  knobIndex.value = idx;
}

onBeforeUnmount(() => {
  stopDrag();
});
</script>

<style scoped>
input[type="number"]::-webkit-outer-spin-button,
input[type="number"]::-webkit-inner-spin-button {
  -webkit-appearance: none;
  margin: 0;
}
input[type="number"] {
  -moz-appearance: textfield;
  appearance: textfield;
}
</style>
