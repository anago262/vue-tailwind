<template>
  <div class="w-full flex flex-col items-center mt-16">
    <div ref="barRef" class="relative w-[500px] h-2 bg-gray-300 rounded-full" @click="onBarClick">
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
  </div>
</template>

<script setup>
import { ref, computed, onBeforeUnmount } from 'vue';

const props = defineProps({
  dotCount: { type: Number, default: 10 },
  dotSize: { type: [Number, String], default: 24 },
  knobSize: { type: [Number, String], default: null },
  knobColor: { type: String, default: '#0ea5e9' }
});

const dotCount = props.dotCount;
const knobIndex = ref(0);
const barRef = ref(null);
const dragging = ref(false);

// ドットの大きさを px 形式で返す
const dotSizePx = computed(() =>
  typeof props.dotSize === 'number' ? `${props.dotSize}px` : props.dotSize
);

// つまみサイズを計算。props.knobSize があればそれを優先、なければ dotSize + 8px
const knobSizePx = computed(() => {
  if (props.knobSize !== null && props.knobSize !== undefined) {
    return typeof props.knobSize === 'number'
      ? `${props.knobSize}px`
      : props.knobSize;
  }
  return `calc(${dotSizePx.value} + 8px)`;
});

function moveKnob(idx) {
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
  const idx = Math.round(ratio * (dotCount - 1));
  knobIndex.value = idx;
}

onBeforeUnmount(() => {
  stopDrag();
});
</script>

<style scoped>
/* transform で中央揃えしているので追加スタイル不要 */
</style>