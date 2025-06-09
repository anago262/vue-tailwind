<template>
  <div class="w-full flex flex-col items-center mt-16">
    <!-- スライダー＋inputを横並び -->
    <div class="flex items-center w-full justify-center space-x-8 mb-4">
      <!-- 左側：スコアinput -->
      <div class="flex flex-col items-center">
        <span class="mb-1 text-lg text-gray-600">スコア</span>
        <input
          type="number"
          class="text-2xl font-bold text-blue-700 border-b-2 border-blue-300 focus:outline-none w-24 text-center"
          :min="dotValues[0]"
          :max="dotValues[dotValues.length - 1]"
          :step="10"
          v-model="inputValue"
          @change="onInputChange"
        />
      </div>
      <!-- 中央：スライダー本体 -->
      <div class="relative w-[500px] h-10 flex items-center">
      <!-- 横棒：左側（濃い青） -->
      <div
        class="absolute top-1/2 left-0 bg-blue-700 rounded-full -translate-y-1/2 z-10"
        :style="{
          width: leftBarWidth,
          height: barHeightPx,
          transition: 'width 0.2s'
        }"
        @click="onBarClick"
      ></div>
      <!-- 横棒：右側（薄い青） -->
      <div
        class="absolute top-1/2 left-0 bg-blue-200 rounded-full -translate-y-1/2 z-0"
        :style="{
          width: '100%',
          height: barHeightPx
        }"
        @click="onBarClick"
      ></div>
      <!-- 動く丸（つまみ） -->
      <div
        class="absolute z-20 transition-all duration-200"
        :style="knobStyle"
        @mousedown="onKnobMouseDown"
        @touchstart.prevent="onKnobTouchStart"
      ></div>
      <!-- 丸（目盛り） -->
      <div class="w-full flex justify-between z-10">
        <template v-for="(value, i) in dotValues" :key="i">
          <div class="flex flex-col items-center">
            <div
              class="bg-white border-2 border-gray-400 rounded-full flex items-center justify-center shadow cursor-pointer"
              :style="{ width: dotSizePx, height: dotSizePx }"
              @click="moveKnob(i)"
            ></div>
            
          </div>
        </template>
      </div>
    </div>
      <!-- 右側：スコア2input -->
      <div class="flex flex-col items-center">
        <span class="mb-1 text-lg text-gray-600">スコア2</span>
        <input
          type="number"
          class="text-2xl font-bold text-pink-700 border-b-2 border-pink-300 focus:outline-none w-24 text-center"
          :min="1"
          :max="99"
          :step="10"
          v-model="inputValue2"
          @change="onInput2Change"
        />
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed, ref, watch, onMounted, onBeforeUnmount } from "vue";

const props = defineProps({
  barHeight: {
    type: [String, Number],
    default: 8 // px
  },
  dotSize: {
    type: [String, Number],
    default: 24 // px
  },
  knobSize: {
    type: [String, Number],
    default: null // dotSize+8がデフォルト
  },
  knobColor: {
    type: String,
    default: "#0ea5e9"
  },
  modelValue: {
    type: Number,
    default: 50
  }
});

const emit = defineEmits(["update:modelValue"]);

const barHeightPx = computed(() =>
  typeof props.barHeight === "number" ? `${props.barHeight}px` : props.barHeight
);
const dotSizePx = computed(() =>
  typeof props.dotSize === "number" ? `${props.dotSize}px` : props.dotSize
);

// スライドバーの幅(px)と目盛り数
const barWidth = 500;
const dotCount = 11;
const dotValues = Array.from({ length: dotCount }, (_, i) => i * 10);

const knobIndex = ref(dotValues.findIndex(v => v === 50));

// modelValueが変わったらknobIndexを同期
watch(
  () => props.modelValue,
  (val) => {
    // 最も近いdotValuesのインデックスを探す
    let idx = dotValues.findIndex(v => v === val);
    if (idx === -1) {
      // 10刻み以外なら最も近い値にスナップ
      let minDiff = Infinity;
      dotValues.forEach((v, i) => {
        const diff = Math.abs(v - val);
        if (diff < minDiff) {
          minDiff = diff;
          idx = i;
        }
      });
    }
    knobIndex.value = idx;
  },
  { immediate: true }
);

// 現在のスコア
const currentScore = computed(() => dotValues[knobIndex.value]);

 // スコア2（100-スコア）
const score2 = computed(() => 100 - currentScore.value);

// input用
const inputValue = ref(currentScore.value);
// スコア2入力用
const inputValue2 = ref(score2.value);

// knobIndexが変わったらinputValueも同期し、emit
watch(knobIndex, () => {
  inputValue.value = currentScore.value;
  inputValue2.value = score2.value;
  emit("update:modelValue", currentScore.value);
});

function onInputChange() {
  let val = Number(inputValue.value);
  if (isNaN(val)) val = dotValues[0];
  val = Math.max(dotValues[0], Math.min(val, dotValues[dotValues.length - 1]));
  // 最も近いdotValuesのインデックスを探す
  let idx = dotValues.findIndex(v => v === val);
  if (idx === -1) {
    // 10刻み以外なら最も近い値にスナップ
    let minDiff = Infinity;
    dotValues.forEach((v, i) => {
      const diff = Math.abs(v - val);
      if (diff < minDiff) {
        minDiff = diff;
        idx = i;
      }
    });
  }
  // 0や100（端）は選択不可
  if (idx <= 0) idx = 1;
  if (idx >= dotValues.length - 1) idx = dotValues.length - 2;
  knobIndex.value = idx;
  inputValue.value = dotValues[idx];
  inputValue2.value = 100 - dotValues[idx];
  emit("update:modelValue", dotValues[idx]);
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
  // 0や100（端）は選択不可
  if (idx <= 0) idx = 1;
  if (idx >= dotValues.length - 1) idx = dotValues.length - 2;
  knobIndex.value = idx;
  inputValue.value = dotValues[idx];
  inputValue2.value = 100 - dotValues[idx];
  emit("update:modelValue", dotValues[idx]);
}

/*
// スライドバーの幅(px)と目盛り数
const barWidth = 500;
const dotCount = 11;
const dotValues = Array.from({ length: dotCount }, (_, i) => i * 10);
*/

// 横棒の左側（濃い青）の幅
const leftBarWidth = computed(() => {
  const percent = (knobIndex.value) / (dotCount - 1) * 100;
  return percent + "%";
});

const knobSizePx = computed(() => {
  if (props.knobSize !== null && props.knobSize !== undefined) {
    return typeof props.knobSize === "number" ? `${props.knobSize}px` : props.knobSize;
  }
  const size = typeof props.dotSize === "number"
    ? props.dotSize + 8
    : parseInt(props.dotSize) + 8;
  return `${size}px`;
});

// つまみの位置計算
const knobStyle = computed(() => {
  const step = (barWidth - Number(dotSizePx.value.replace('px', ''))) / (dotCount - 1);
  const left = knobIndex.value * step;
  const knobSizeNum = Number(knobSizePx.value.replace('px', ''));
  const dotSizeNum = Number(dotSizePx.value.replace('px', ''));
  return {
    top: `calc(50% - ${knobSizeNum / 2}px)`,
    left: `calc(${left + dotSizeNum / 2 - knobSizeNum / 2}px)`,
    width: knobSizePx.value,
    height: knobSizePx.value,
    background: props.knobColor,
    border: `3px solid ${props.knobColor}`,
    borderRadius: "50%",
    cursor: "pointer"
  };
});

/**
 * 目盛りクリックでつまみを移動
 */
function moveKnob(idx) {
  // 0や100（端）は選択不可: 端の丸をクリックしても何もしない
  if (idx <= 0 || idx >= dotValues.length - 1) return;
  knobIndex.value = idx;
  emit("update:modelValue", dotValues[idx]);
}

/**
 * 横棒クリックで最寄りの目盛りに移動
 */
function onBarClick(e) {
  const bar = e.currentTarget;
  const rect = bar.getBoundingClientRect();
  let x = e.clientX - rect.left;
  x = Math.max(0, Math.min(x, barWidth));
  const step = (barWidth - parseInt(props.dotSize)) / (dotCount - 1);
  let idx = Math.round(x / step);
  idx = Math.max(0, Math.min(idx, dotCount - 1));
  // 0や100（端）は選択不可
  if (idx <= 0) idx = 1;
  if (idx >= dotValues.length - 1) idx = dotValues.length - 2;
  knobIndex.value = idx;
  inputValue.value = dotValues[idx];
  inputValue2.value = 100 - dotValues[idx];
  emit("update:modelValue", dotValues[idx]);
}

// ドラッグ用
const dragging = ref(false);

function onKnobMouseDown(e) {
  dragging.value = true;
  window.addEventListener("mousemove", onMouseMove);
  window.addEventListener("mouseup", onMouseUp);
}

function onMouseMove(e) {
  if (!dragging.value) return;
  updateKnobByClientX(e.clientX);
}

function onMouseUp() {
  dragging.value = false;
  window.removeEventListener("mousemove", onMouseMove);
  window.removeEventListener("mouseup", onMouseUp);
}

// タッチ対応
function onKnobTouchStart(e) {
  dragging.value = true;
  window.addEventListener("touchmove", onTouchMove, { passive: false });
  window.addEventListener("touchend", onTouchEnd);
}

function onTouchMove(e) {
  if (!dragging.value) return;
  if (e.touches && e.touches.length > 0) {
    updateKnobByClientX(e.touches[0].clientX);
  }
}

function onTouchEnd() {
  dragging.value = false;
  window.removeEventListener("touchmove", onTouchMove);
  window.removeEventListener("touchend", onTouchEnd);
}

// マウス座標からknobIndexを計算
function updateKnobByClientX(clientX) {
  const bar = document.querySelector(".relative.w-\\[500px\\].h-10.flex.items-center");
  if (!bar) return;
  const rect = bar.getBoundingClientRect();
  let x = clientX - rect.left;
  x = Math.max(0, Math.min(x, barWidth));
  const step = (barWidth - parseInt(props.dotSize)) / (dotCount - 1);
  let idx = Math.round(x / step);
  idx = Math.max(0, Math.min(idx, dotCount - 1));
  // 0や100（端）は選択不可
  if (idx <= 0) idx = 1;
  if (idx >= dotValues.length - 1) idx = dotValues.length - 2;
  knobIndex.value = idx;
  emit("update:modelValue", dotValues[idx]);
}

// クリーンアップ
onBeforeUnmount(() => {
  window.removeEventListener("mousemove", onMouseMove);
  window.removeEventListener("mouseup", onMouseUp);
  window.removeEventListener("touchmove", onTouchMove);
  window.removeEventListener("touchend", onTouchEnd);
});
</script>
<style scoped>
/* type="number"のスピンボタン（上下矢印）を非表示にする */
input[type="number"]::-webkit-outer-spin-button,
input[type="number"]::-webkit-inner-spin-button {
  -webkit-appearance: none;
  margin: 0;
}
input[type="number"] {
  -moz-appearance: textfield; /* Firefox */
  appearance: textfield;
}
</style>
