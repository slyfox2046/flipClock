<script lang="ts">
import cssText from "data-text:~/contents/clock-overlay.css"
import type { PlasmoCSConfig, PlasmoGetStyle } from "plasmo"

export const config: PlasmoCSConfig = {
  matches: ["<all_urls>"]
}

const getStyle: PlasmoGetStyle = () => {
  const style = document.createElement("style")
  style.textContent = cssText
  return style
}

export default {
  plasmo: {
    getStyle
  }
}
</script>

<script setup lang="ts">
import { computed, onMounted, onUnmounted, reactive, ref } from "vue"
import FlipDigit from "~/contents/FlipDigit.vue"

const STORAGE_KEY = "fclock_settings"

type Settings = {
  enabled: boolean
  use24h: boolean
  x: number
  y: number
}

const defaults: Settings = {
  enabled: true,
  use24h: true,
  x: 24,
  y: 24
}

const settings = reactive<Settings>({ ...defaults })
const visible = computed(() => settings.enabled)

const hoursTens = ref("0")
const hoursOnes = ref("0")
const minTens = ref("0")
const minOnes = ref("0")
const secTens = ref("0")
const secOnes = ref("0")
const ampm = ref("")
const colonBright = ref(true)

let tickTimer: ReturnType<typeof setInterval> | null = null
let colonTimer: ReturnType<typeof setInterval> | null = null

function pad2(n: number) {
  return String(n).padStart(2, "0")
}

function applyTime() {
  const now = new Date()
  let h = now.getHours()
  const m = now.getMinutes()
  const s = now.getSeconds()

  if (!settings.use24h) {
    ampm.value = h >= 12 ? "PM" : "AM"
    h = h % 12
    if (h === 0) h = 12
  } else {
    ampm.value = ""
  }

  const hs = pad2(settings.use24h ? now.getHours() : h)
  hoursTens.value = hs[0]
  hoursOnes.value = hs[1]
  const ms = pad2(m)
  minTens.value = ms[0]
  minOnes.value = ms[1]
  const ss = pad2(s)
  secTens.value = ss[0]
  secOnes.value = ss[1]
}

function mergeSettings(raw: Partial<Settings> | undefined) {
  if (!raw) return
  if (typeof raw.enabled === "boolean") settings.enabled = raw.enabled
  if (typeof raw.use24h === "boolean") settings.use24h = raw.use24h
  if (typeof raw.x === "number") settings.x = raw.x
  if (typeof raw.y === "number") settings.y = raw.y
}

async function loadSettings() {
  const data = await chrome.storage.local.get(STORAGE_KEY)
  mergeSettings(data[STORAGE_KEY] as Partial<Settings> | undefined)
}

function onStorageChange(
  changes: Record<string, chrome.storage.StorageChange>,
  area: string
) {
  if (area !== "local" || !changes[STORAGE_KEY]) return
  mergeSettings(changes[STORAGE_KEY].newValue as Partial<Settings> | undefined)
}

const rootStyle = computed(() => ({
  left: `${settings.x}px`,
  top: `${settings.y}px`,
  position: "fixed" as const
}))

/* ---------- 拖拽 ---------- */
const dragging = ref(false)
let dragStartX = 0
let dragStartY = 0
let originX = 0
let originY = 0

function onDragStart(e: MouseEvent) {
  if (e.button !== 0) return
  dragging.value = true
  dragStartX = e.clientX
  dragStartY = e.clientY
  originX = settings.x
  originY = settings.y
  e.preventDefault()
}

function onDragMove(e: MouseEvent) {
  if (!dragging.value) return
  const dx = e.clientX - dragStartX
  const dy = e.clientY - dragStartY
  settings.x = Math.max(0, originX + dx)
  settings.y = Math.max(0, originY + dy)
}

function onDragEnd() {
  if (!dragging.value) return
  dragging.value = false
  void chrome.storage.local.set({
    [STORAGE_KEY]: {
      enabled: settings.enabled,
      use24h: settings.use24h,
      x: settings.x,
      y: settings.y
    }
  })
}

onMounted(() => {
  void loadSettings()
  applyTime()
  tickTimer = setInterval(applyTime, 250)
  colonTimer = setInterval(() => {
    colonBright.value = !colonBright.value
  }, 500)
  chrome.storage.onChanged.addListener(onStorageChange)
  window.addEventListener("mousemove", onDragMove)
  window.addEventListener("mouseup", onDragEnd)
})

onUnmounted(() => {
  if (tickTimer) clearInterval(tickTimer)
  if (colonTimer) clearInterval(colonTimer)
  chrome.storage.onChanged.removeListener(onStorageChange)
  window.removeEventListener("mousemove", onDragMove)
  window.removeEventListener("mouseup", onDragEnd)
})
</script>

<template>
  <div v-show="visible" class="fclock-root" :style="rootStyle">
    <div class="fclock-panel">
      <div
        class="fclock-drag"
        title="拖动移动"
        @mousedown="onDragStart">
        <span class="fclock-drag-dots">
          <span /><span /><span />
        </span>
      </div>
      <div class="fclock-row">
        <FlipDigit :digit="hoursTens" />
        <FlipDigit :digit="hoursOnes" />
        <span class="fclock-sep" :class="{ 'fclock-sep-dim': !colonBright }">:</span>
        <FlipDigit :digit="minTens" />
        <FlipDigit :digit="minOnes" />
        <span class="fclock-sep" :class="{ 'fclock-sep-dim': !colonBright }">:</span>
        <FlipDigit :digit="secTens" />
        <FlipDigit :digit="secOnes" />
        <span v-if="ampm" class="fclock-ampm">{{ ampm }}</span>
      </div>
    </div>
  </div>
</template>
