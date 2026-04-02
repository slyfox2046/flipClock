<template>
  <div class="newtab-container">
    <div class="clock-wrapper">
      <div class="clock-display">
        <FlipDigit :digit="hoursTens" :height="350" :fontSize="245" />
        <FlipDigit :digit="hoursOnes" :height="350" :fontSize="245" />
        <div class="separator"></div>
        <FlipDigit :digit="minTens" :height="350" :fontSize="245" />
        <FlipDigit :digit="minOnes" :height="350" :fontSize="245" />
        <div class="separator"></div>
        <FlipDigit :digit="secTens" :height="350" :fontSize="245" />
        <FlipDigit :digit="secOnes" :height="350" :fontSize="245" />
      </div>
      <div class="date-display">
        {{ dateString }}
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { onMounted, onUnmounted, ref } from "vue"

import FlipDigit from "~/components/FlipDigit.vue"

import "~/tabs/newtab.css"

const hoursTens = ref("0")
const hoursOnes = ref("0")
const minTens = ref("0")
const minOnes = ref("0")
const secTens = ref("0")
const secOnes = ref("0")
const dateString = ref("")

let tickTimer: ReturnType<typeof setInterval> | null = null

function pad2(n: number) {
  return String(n).padStart(2, "0")
}

function applyTime() {
  const now = new Date()
  const year = now.getFullYear()
  const month = now.getMonth() + 1
  const day = now.getDate()
  const h = now.getHours()
  const m = now.getMinutes()
  const s = now.getSeconds()

  // 小时（24小时制）
  const hs = pad2(h)
  hoursTens.value = hs[0]
  hoursOnes.value = hs[1]

  // 分钟
  const ms = pad2(m)
  minTens.value = ms[0]
  minOnes.value = ms[1]

  // 秒
  const ss = pad2(s)
  secTens.value = ss[0]
  secOnes.value = ss[1]

  // 日期字符串
  const weekdays = ["SUN", "MON", "TUE", "WED", "THU", "FRI", "SAT"]
  const weekday = weekdays[now.getDay()]
  dateString.value = `${year}/${pad2(month)}/${pad2(day)} ${weekday}`
}

onMounted(() => {
  applyTime()
  tickTimer = setInterval(applyTime, 250)
})

onUnmounted(() => {
  if (tickTimer) clearInterval(tickTimer)
})
</script>

<style>
body {
  margin: 0;
  padding: 0;
  overflow: hidden;
}
</style>

<style scoped>
.newtab-container {
  width: 100vw;
  height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  /* 参考图：近黑炭灰底 */
  background: #0f0f10;
  font-family:
    system-ui,
    -apple-system,
    sans-serif;
  overflow: hidden;
}

.clock-wrapper {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 40px;
}

.clock-display {
  display: flex;
  align-items: center;
  /* 参考图：数字块之间小间距 */
  gap: 14px;
}

.separator {
  width: 20px;
}

.date-display {
  font-size: 22px;
  font-weight: 400;
  color: rgba(255, 255, 255, 0.38);
  text-align: center;
  letter-spacing: 0.14em;
  font-variant-numeric: tabular-nums;
}
</style>
