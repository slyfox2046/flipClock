<template>
  <div
    class="fclock-digit"
    :style="{ '--fclock-digit-h': height + 'px', '--fclock-fs': fontSize + 'px' }">
    <div class="fclock-digit-inner">
      <!-- 静态层：显示当前数字的上下两半 -->
      <div class="fclock-static fclock-top">
        <span>{{ digit }}</span>
      </div>
      <div class="fclock-static fclock-bottom">
        <span>{{ digit }}</span>
      </div>
      <!-- 翻页层：从旧数字翻到新数字 -->
      <div
        class="fclock-flap fclock-flap-top"
        :class="{ 'fclock-flip': flipping }"
        >
        <span>{{ prevDigit }}</span>
      </div>
      <div
        class="fclock-flap fclock-flap-bottom"
        :class="{ 'fclock-flip': flipping }"
        @animationend="onBottomEnd">
        <span>{{ digit }}</span>
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
