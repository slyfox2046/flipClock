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
        <template v-if="weatherReady">
          <template v-if="locationName">
            <span class="weather-sep">·</span>
            <span class="weather-location">{{ locationName }}</span>
          </template>
          <span class="weather-sep">·</span>
          <span class="weather-temp">{{ temperature }}°C</span>
          <span class="weather-sep">·</span>
          <span class="weather-desc">{{ weatherDesc }}</span>

        </template>
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

const temperature = ref("")
const weatherDesc = ref("")
const locationName = ref("")
const weatherReady = ref(false)

// WMO 天气代码 → 中文描述
const weatherCodeMap: Record<number, string> = {
  0: "晴天",
  1: "晴",
  2: "多云",
  3: "阴天",
  45: "雾",
  48: "冻雾",
  51: "轻度毛毛雨",
  53: "中度毛毛雨",
  55: "重度毛毛雨",
  56: "轻度冻毛毛雨",
  57: "重度冻毛毛雨",
  61: "小雨",
  63: "中雨",
  65: "大雨",
  66: "轻度冻雨",
  67: "重度冻雨",
  71: "小雪",
  73: "中雪",
  75: "大雪",
  77: "雪粒",
  80: "小阵雨",
  81: "中阵雨",
  82: "强阵雨",
  85: "小雪阵雨",
  86: "大雪阵雨",
  95: "雷暴",
  96: "雷暴伴小冰雹",
  99: "雷暴伴大冰雹",
}

function getPosition(): Promise<GeolocationCoordinates> {
  return new Promise((resolve, reject) => {
    navigator.geolocation.getCurrentPosition(
      (pos) => resolve(pos.coords),
      reject,
      { timeout: 5000 }
    )
  })
}

async function fetchWeather() {
  try {
    const coords = await getPosition()
    const { latitude, longitude } = coords
    console.log(`经纬度：latitude=${latitude}, longitude=${longitude}`)

    // 反向地理编码，获取中文地名（高德 Web 服务）
    const geoRes = await fetch(
      `https://amap.com/service/regeo?longitude=${longitude}&latitude=${latitude}`
    )
    const geoData = await geoRes.json()
    const d = geoData.data ?? {}
    // 省份 + 区县，例如：上海市浦东新区
    locationName.value = `${d.province ?? ""}${d.district ?? ""}`
    console.log(`位置：${locationName.value}`, d)

    const res = await fetch(
      `https://api.open-meteo.com/v1/forecast?latitude=${latitude}&longitude=${longitude}&current_weather=true&timezone=Asia/Shanghai`
    )
    const data = await res.json()
    const cw = data.current_weather
    temperature.value = String(cw.temperature)
    weatherDesc.value = weatherCodeMap[cw.weathercode] ?? "--"
    weatherReady.value = true
  } catch {
    // 定位失败或网络异常时不显示天气
  }
}

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
  fetchWeather()
  // 每30分钟刷新一次天气
  setInterval(fetchWeather, 30 * 60 * 1000)
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

.weather-sep {
  opacity: 0.4;
  margin: 0 6px;
}
</style>
