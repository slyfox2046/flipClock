<template>
  <div
    class="fclock-digit"
    :style="{ '--fclock-digit-h': height + 'px', '--fclock-fs': fontSize + 'px' }">
    <div class="fclock-digit-inner">
      <!-- 静态层：用「整格高度 + overflow 裁一半」对齐上下笔画，避免 translateY 与 3D transform 叠加重影 -->
      <div class="fclock-static fclock-top">
        <span class="fclock-glyph fclock-glyph-top">{{ digit }}</span>
      </div>
      <div class="fclock-static fclock-bottom">
        <span class="fclock-glyph fclock-glyph-bottom">{{ digit }}</span>
      </div>
      <div
        class="fclock-flap fclock-flap-top"
        :class="{ 'fclock-flip': flipping }">
        <span class="fclock-glyph fclock-glyph-top">{{ prevDigit }}</span>
      </div>
      <div
        class="fclock-flap fclock-flap-bottom"
        :class="{ 'fclock-flip': flipping }"
        @animationend="onBottomEnd">
        <span class="fclock-glyph fclock-glyph-bottom">{{ digit }}</span>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, watch } from "vue"

const props = withDefaults(
  defineProps<{
    digit: string
    height?: number
    fontSize?: number
  }>(),
  {
    height: 56,
    fontSize: 36
  }
)

const prevDigit = ref(props.digit)
const flipping = ref(false)

function startFlip(from: string, to: string) {
  if (from === to) return
  prevDigit.value = from
  flipping.value = true
}

function onBottomEnd() {
  flipping.value = false
  prevDigit.value = props.digit
}

watch(
  () => props.digit,
  (next, prev) => {
    if (prev === undefined) {
      prevDigit.value = next
      return
    }
    if (next === prev) return
    if (flipping.value) return
    startFlip(prev, next)
  }
)
</script>

<style scoped>
.fclock-digit {
  --fclock-digit-h: 56px;
  --fclock-fs: 36px;
  /* 与卡片高度成比例的圆角，大屏也保持「略方带圆」 */
  --fclock-tile-r: clamp(6px, calc(var(--fclock-digit-h) * 0.034), 14px);
  position: relative;
  width: calc(var(--fclock-fs) * 0.78);
  height: var(--fclock-digit-h);
  font-size: var(--fclock-fs);
  perspective: calc(var(--fclock-digit-h) * 2.5);
  -webkit-font-smoothing: antialiased;
  /* 参考图：整块略浮起、轻外阴影 */
  border-radius: var(--fclock-tile-r);
  box-shadow:
    0 calc(var(--fclock-digit-h) * 0.018) calc(var(--fclock-digit-h) * 0.06)
      rgba(0, 0, 0, 0.55),
    0 calc(var(--fclock-digit-h) * 0.006) calc(var(--fclock-digit-h) * 0.02)
      rgba(0, 0, 0, 0.35);
}

.fclock-digit-inner {
  position: absolute;
  inset: 0;
  border-radius: var(--fclock-tile-r);
  overflow: hidden;
  /* 顶缘微高光 + 内阴影增加厚度 */
  box-shadow:
    inset 0 1px 0 rgba(255, 255, 255, 0.1),
    inset 0 calc(var(--fclock-digit-h) * -0.14) calc(var(--fclock-digit-h) * 0.2)
      rgba(0, 0, 0, 0.35);
}

/* 数字笔画：占满整卡高度，用 line-height 垂直居中，再由上下层各裁一半 */
.fclock-glyph {
  position: absolute;
  left: 0;
  right: 0;
  height: var(--fclock-digit-h);
  line-height: var(--fclock-digit-h);
  text-align: center;
  font-weight: 700;
  font-variant-numeric: tabular-nums;
  user-select: none;
  color: #ffffff;
  text-shadow: 0 1px 0 rgba(0, 0, 0, 0.35);
  transform: none;
}

.fclock-glyph-top {
  top: 0;
}

.fclock-glyph-bottom {
  bottom: 0;
}

.fclock-static {
  position: absolute;
  left: 0;
  right: 0;
  overflow: hidden;
}

/* 上半：自上而下略变暗，模拟物理翻页片上沿受光 */
.fclock-static.fclock-top {
  background: linear-gradient(
    180deg,
    #4a5059 0%,
    #3d424b 28%,
    #32363e 65%,
    #2b2f36 100%
  );
}

/* 下半：整体更深，与上半形成层次 */
.fclock-static.fclock-bottom {
  background: linear-gradient(
    180deg,
    #25282f 0%,
    #1e2128 45%,
    #181a20 100%
  );
}

.fclock-top {
  top: 0;
  height: 50%;
  border-bottom: 1px solid rgba(0, 0, 0, 0.65);
  z-index: 0;
}

.fclock-bottom {
  bottom: 0;
  height: 50%;
  z-index: 0;
}

.fclock-flap {
  position: absolute;
  left: 0;
  right: 0;
  height: 50%;
  overflow: hidden;
  backface-visibility: hidden;
  transform-style: preserve-3d;
  pointer-events: none;
}

.fclock-flap.fclock-flap-top {
  background: linear-gradient(
    180deg,
    #434851 0%,
    #383c45 35%,
    #2f333b 100%
  );
}

.fclock-flap.fclock-flap-bottom {
  background: linear-gradient(
    180deg,
    #282b32 0%,
    #20242b 50%,
    #1a1d23 100%
  );
}

.fclock-flap-top {
  top: 0;
  transform-origin: bottom center;
  border-bottom: 1px solid rgba(0, 0, 0, 0.6);
  z-index: 2;
}

.fclock-flap-bottom {
  bottom: 0;
  transform-origin: top center;
  z-index: 1;
  transform: rotateX(90deg);
}

.fclock-flap-top.fclock-flip {
  animation: fclock-flip-top 0.32s ease-in forwards;
}

.fclock-flap-bottom.fclock-flip {
  animation: fclock-flip-bottom 0.32s ease-out forwards;
}

@keyframes fclock-flip-top {
  from {
    transform: rotateX(0deg);
  }
  to {
    transform: rotateX(-90deg);
  }
}

@keyframes fclock-flip-bottom {
  from {
    transform: rotateX(90deg);
  }
  to {
    transform: rotateX(0deg);
  }
}
</style>
