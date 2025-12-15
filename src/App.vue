<script setup>
import { ref, computed, watch } from 'vue'

import PaletteBar from './components/PaletteBar.vue'
import PreviewMockup from './components/PreviewMockup.vue'

// ---- Константы ----
const PALETTE_SIZES = [3, 5, 7]

const FORMAT_OPTIONS = [
  { value: 'hex', label: 'HEX' },
  { value: 'rgb', label: 'RGB' },
]

const HARMONY_TYPES = [
  { value: 'random', label: 'Случайная' },
  { value: 'analogous', label: 'Аналогичная' },
  { value: 'monochromatic', label: 'Монохромная' },
  { value: 'triad', label: 'Триада' },
  { value: 'complementary', label: 'Комплементарная' },
]

const MOODS = [
  { value: 'none', label: 'Без настроения' },
  { value: 'calm', label: 'Спокойная' },
  { value: 'energetic', label: 'Энергичная' },
  { value: 'professional', label: 'Профессиональная' },
]

// ---- Вспомогательные функции для работы с цветом ----
function getRandomInt(min, max) {
  return Math.floor(Math.random() * (max - min + 1)) + min
}

function randomHexColor() {
  const r = getRandomInt(0, 255)
  const g = getRandomInt(0, 255)
  const b = getRandomInt(0, 255)
  return rgbToHex(r, g, b)
}

function rgbToHex(r, g, b) {
  const toHex = (n) => n.toString(16).padStart(2, '0')
  return `#${toHex(r)}${toHex(g)}${toHex(b)}`.toUpperCase()
}

function hexToRgb(hex) {
  const clean = hex.replace('#', '')
  const bigint = parseInt(clean, 16)
  const r = (bigint >> 16) & 255
  const g = (bigint >> 8) & 255
  const b = bigint & 255
  return { r, g, b }
}

function adjustHue(hex, delta) {
  const { r, g, b } = hexToRgb(hex)
  // перевод в HSL
  const rNorm = r / 255
  const gNorm = g / 255
  const bNorm = b / 255
  const max = Math.max(rNorm, gNorm, bNorm)
  const min = Math.min(rNorm, gNorm, bNorm)
  let h = 0
  const l = (max + min) / 2
  const d = max - min
  let s = 0

  if (d !== 0) {
    s = d / (1 - Math.abs(2 * l - 1))
    switch (max) {
      case rNorm:
        h = ((gNorm - bNorm) / d) % 6
        break
      case gNorm:
        h = (bNorm - rNorm) / d + 2
        break
      case bNorm:
        h = (rNorm - gNorm) / d + 4
        break
    }
    h *= 60
  }

  h = (h + delta + 360) % 360

  // HSL -> RGB
  const c = (1 - Math.abs(2 * l - 1)) * s
  const x = c * (1 - Math.abs(((h / 60) % 2) - 1))
  const m = l - c / 2
  let r1 = 0
  let g1 = 0
  let b1 = 0

  if (h < 60) {
    r1 = c
    g1 = x
    b1 = 0
  } else if (h < 120) {
    r1 = x
    g1 = c
    b1 = 0
  } else if (h < 180) {
    r1 = 0
    g1 = c
    b1 = x
  } else if (h < 240) {
    r1 = 0
    g1 = x
    b1 = c
  } else if (h < 300) {
    r1 = x
    g1 = 0
    b1 = c
  } else {
    r1 = c
    g1 = 0
    b1 = x
  }

  return rgbToHex(
    Math.round((r1 + m) * 255),
    Math.round((g1 + m) * 255),
    Math.round((b1 + m) * 255),
  )
}

function applyMood(baseHex, mood) {
  const { r, g, b } = hexToRgb(baseHex)
  let factor = 1
  let shift = 0

  switch (mood) {
    case 'calm':
      factor = 0.9
      shift = -10
      break
    case 'energetic':
      factor = 1.1
      shift = 10
      break
    case 'professional':
      factor = 0.95
      shift = 0
      break
    default:
      return baseHex
  }

  const clamp = (v) => Math.min(255, Math.max(0, v))
  const nr = clamp(r * factor + shift)
  const ng = clamp(g * factor + shift)
  const nb = clamp(b * factor + shift)
  return rgbToHex(Math.round(nr), Math.round(ng), Math.round(nb))
}

function computeContrast(hex1, hex2) {
  const rgb1 = hexToRgb(hex1)
  const rgb2 = hexToRgb(hex2)

  const luminance = ({ r, g, b }) => {
    const channel = (c) => {
      const v = c / 255
      return v <= 0.03928 ? v / 12.92 : Math.pow((v + 0.055) / 1.055, 2.4)
    }
    return 0.2126 * channel(r) + 0.7152 * channel(g) + 0.0722 * channel(b)
  }

  const L1 = luminance(rgb1)
  const L2 = luminance(rgb2)
  const ratio = (Math.max(L1, L2) + 0.05) / (Math.min(L1, L2) + 0.05)

  let level = 'Недостаточно'
  if (ratio >= 7) {
    level = 'AAA'
  } else if (ratio >= 4.5) {
    level = 'AA'
  }

  return {
    ratio: ratio.toFixed(2),
    level,
  }
}

// ---- Состояние приложения ----
const paletteSize = ref(5)
const colorFormat = ref('hex')
const palette = ref(
  Array.from({ length: paletteSize.value }).map(() => ({
    color: randomHexColor(),
    locked: false,
  })),
)

const harmonyType = ref('random')
const baseColor = ref('#FF6B6B')
const mood = ref('none')
const isDarkPreview = ref(false)

// ---- Загрузка/сохранение состояния ----
const STORAGE_KEY = 'vue_palette_generator_state'

function loadFromStorage() {
  try {
    const raw = localStorage.getItem(STORAGE_KEY)
    if (!raw) return
    const data = JSON.parse(raw)
    if (Array.isArray(data.palette) && data.palette.length) {
      palette.value = data.palette
    }
    if (PALETTE_SIZES.includes(data.paletteSize)) {
      paletteSize.value = data.paletteSize
    }
    if (FORMAT_OPTIONS.some((f) => f.value === data.colorFormat)) {
      colorFormat.value = data.colorFormat
    }
    if (HARMONY_TYPES.some((h) => h.value === data.harmonyType)) {
      harmonyType.value = data.harmonyType
    }
    if (MOODS.some((m) => m.value === data.mood)) {
      mood.value = data.mood
    }
    if (typeof data.isDarkPreview === 'boolean') {
      isDarkPreview.value = data.isDarkPreview
    }
    if (typeof data.baseColor === 'string') {
      baseColor.value = data.baseColor
    }
  } catch (e) {
    console.error('Не удалось загрузить состояние палитры:', e)
  }
}

function saveToStorage() {
  try {
    const data = {
      palette: palette.value,
      paletteSize: paletteSize.value,
      colorFormat: colorFormat.value,
      harmonyType: harmonyType.value,
      baseColor: baseColor.value,
      mood: mood.value,
      isDarkPreview: isDarkPreview.value,
    }
    localStorage.setItem(STORAGE_KEY, JSON.stringify(data))
  } catch (e) {
    console.error('Не удалось сохранить состояние палитры:', e)
  }
}

watch(
  [palette, paletteSize, colorFormat, harmonyType, baseColor, mood, isDarkPreview],
  saveToStorage,
  { deep: true },
)

// ---- Генерация палитр ----
function generatePalette() {
  let base = baseColor.value || randomHexColor()
  if (mood.value !== 'none') {
    base = applyMood(base, mood.value)
  }

  const newPalette = [...palette.value]
  const needed = paletteSize.value

  // Сначала гарантируем нужную длину
  while (newPalette.length < needed) {
    newPalette.push({ color: randomHexColor(), locked: false })
  }
  if (newPalette.length > needed) {
    newPalette.length = needed
  }

  const unlockedIndices = newPalette
    .map((c, index) => (c.locked ? null : index))
    .filter((v) => v !== null)

  if (harmonyType.value === 'random') {
    unlockedIndices.forEach((i) => {
      newPalette[i] = {
        ...newPalette[i],
        color: randomHexColor(),
      }
    })
  } else if (harmonyType.value === 'analogous') {
    const step = 20
    unlockedIndices.forEach((i, idx) => {
      const delta = (idx - Math.floor(needed / 2)) * step
      newPalette[i] = {
        ...newPalette[i],
        color: adjustHue(base, delta),
      }
    })
  } else if (harmonyType.value === 'monochromatic') {
    unlockedIndices.forEach((i) => {
      // небольшое случайное изменение яркости
      const randomDelta = getRandomInt(-15, 15)
      newPalette[i] = {
        ...newPalette[i],
        color: applyMood(base, randomDelta > 0 ? 'energetic' : 'calm'),
      }
    })
  } else if (harmonyType.value === 'triad') {
    const hues = [0, 120, 240]
    unlockedIndices.forEach((i, idx) => {
      const delta = hues[idx % 3]
      newPalette[i] = {
        ...newPalette[i],
        color: adjustHue(base, delta),
      }
    })
  } else if (harmonyType.value === 'complementary') {
    unlockedIndices.forEach((i, idx) => {
      const delta = idx % 2 === 0 ? 0 : 180
      newPalette[i] = {
        ...newPalette[i],
        color: adjustHue(base, delta),
      }
    })
  }

  palette.value = newPalette
}

function toggleLock(index) {
  palette.value = palette.value.map((c, i) =>
    i === index
      ? {
          ...c,
          locked: !c.locked,
        }
      : c,
  )
}

async function copyColor(index) {
  const item = palette.value[index]
  if (!item) return

  const text =
    colorFormat.value === 'hex'
      ? item.color
      : (() => {
          const { r, g, b } = hexToRgb(item.color)
          return `rgb(${r}, ${g}, ${b})`
        })()

  try {
    await navigator.clipboard.writeText(text)
    alert(`Цвет ${text} скопирован в буфер обмена`)
  } catch (e) {
    console.error('Не удалось скопировать цвет:', e)
  }
}

const formattedPalette = computed(() =>
  palette.value.map((item) => {
    if (colorFormat.value === 'hex') {
      return {
        ...item,
        label: item.color,
      }
    }
    const { r, g, b } = hexToRgb(item.color)
    return {
      ...item,
      label: `rgb(${r}, ${g}, ${b})`,
    }
  }),
)

const contrastInfo = computed(() => {
  if (!palette.value.length) return null
  const first = palette.value[0].color
  const last = palette.value[palette.value.length - 1].color
  return computeContrast(first, last)
})

// начальная загрузка
loadFromStorage()
</script>

<template>
  <div id="app" class="app">
    <header class="app-header">
      <h1>Генератор цветовых палитр</h1>
      <p>Практики 27–28: Vue 3 + Vite, реактивность и работа с состоянием</p>
    </header>

    <main class="app-main">
      <section class="controls-section">
        <div class="controls-group">
          <h2>Основные настройки</h2>
          <div class="controls-row">
            <label class="control">
              <span>Количество цветов</span>
              <select v-model.number="paletteSize">
                <option v-for="size in PALETTE_SIZES" :key="size" :value="size">
                  {{ size }}
                </option>
              </select>
            </label>

            <label class="control">
              <span>Формат</span>
              <select v-model="colorFormat">
                <option v-for="format in FORMAT_OPTIONS" :key="format.value" :value="format.value">
                  {{ format.label }}
                </option>
              </select>
            </label>

            <label class="control">
              <span>Базовый цвет</span>
              <input v-model="baseColor" type="color" class="color-input" />
            </label>
          </div>
        </div>

        <div class="controls-group">
          <h2>Тип палитры и настроение</h2>
          <div class="controls-row">
            <label class="control">
              <span>Тип палитры</span>
              <select v-model="harmonyType">
                <option v-for="h in HARMONY_TYPES" :key="h.value" :value="h.value">
                  {{ h.label }}
                </option>
              </select>
            </label>

            <label class="control">
              <span>Настроение</span>
              <select v-model="mood">
                <option v-for="m in MOODS" :key="m.value" :value="m.value">
                  {{ m.label }}
                </option>
              </select>
            </label>

            <button class="primary-button" type="button" @click="generatePalette">
              Случайная палитра
            </button>
          </div>
        </div>

        <div v-if="contrastInfo" class="controls-group contrast-info">
          <h2>Контрастность между крайними цветами</h2>
          <p>
            Коэффициент контраста:
            <strong>{{ contrastInfo.ratio }} : 1</strong>
            (уровень:
            <strong>{{ contrastInfo.level }}</strong
            >)
          </p>
        </div>
      </section>

      <section class="palette-section">
        <h2>Текущая палитра</h2>
        <PaletteBar
          :colors="formattedPalette"
          @toggle-lock="toggleLock"
          @copy-color="copyColor"
        />
      </section>

      <section class="preview-section">
        <div class="preview-header">
          <h2>Превью интерфейса</h2>
          <label class="toggle">
            <input v-model="isDarkPreview" type="checkbox" />
            <span>Тёмный фон</span>
          </label>
        </div>
        <PreviewMockup :palette="palette" :dark="isDarkPreview" />
      </section>
    </main>

    <footer class="app-footer">
      <p>Vue 3 + Vite • Практика 27–28 • Генератор цветовых палитр</p>
    </footer>
  </div>
</template>

<style scoped>
.app {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  background-color: #f5f5f5;
  color: #333;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu,
    Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
}

.app-header {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: #fff;
  padding: 2rem;
  text-align: center;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.15);
}

.app-header h1 {
  font-size: 2.4rem;
  margin-bottom: 0.5rem;
}

.app-header p {
  opacity: 0.9;
  font-size: 1rem;
}

.app-main {
  flex: 1;
  padding: 1.5rem;
  max-width: 1200px;
  margin: 0 auto;
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}

.controls-section {
  background-color: #fff;
  border-radius: 16px;
  padding: 1.5rem;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.08);
  display: flex;
  flex-direction: column;
  gap: 1.25rem;
}

.controls-group h2 {
  font-size: 1.1rem;
  margin-bottom: 0.5rem;
}

.controls-row {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(180px, 1fr));
  gap: 1rem;
  align-items: flex-end;
}

.control {
  display: flex;
  flex-direction: column;
  gap: 0.35rem;
  font-size: 0.9rem;
}

.control span {
  font-weight: 500;
  color: #555;
}

select,
input[type='color'] {
  padding: 0.5rem 0.75rem;
  border-radius: 8px;
  border: 1px solid #d1d5db;
  background-color: #f9fafb;
  font-size: 0.95rem;
}

.color-input {
  padding: 0;
  height: 2.4rem;
}

.primary-button {
  padding: 0.75rem 1.5rem;
  border-radius: 999px;
  border: none;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: #fff;
  font-weight: 600;
  cursor: pointer;
  box-shadow: 0 10px 20px rgba(102, 126, 234, 0.35);
  transition: transform 0.15s ease, box-shadow 0.15s ease, opacity 0.15s ease;
  justify-self: flex-end;
}

.primary-button:hover {
  transform: translateY(-1px);
  box-shadow: 0 14px 30px rgba(102, 126, 234, 0.4);
  opacity: 0.95;
}

.primary-button:active {
  transform: translateY(0);
  box-shadow: 0 8px 16px rgba(102, 126, 234, 0.3);
}

.contrast-info p {
  font-size: 0.9rem;
}

.palette-section,
.preview-section {
  background-color: #fff;
  border-radius: 16px;
  padding: 1.5rem;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.06);
}

.palette-section h2,
.preview-section h2 {
  font-size: 1.2rem;
  margin-bottom: 1rem;
}

.preview-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 1rem;
  margin-bottom: 1rem;
}

.toggle {
  display: inline-flex;
  align-items: center;
  gap: 0.5rem;
  font-size: 0.9rem;
}

.app-footer {
  padding: 1rem;
  text-align: center;
  font-size: 0.85rem;
  color: #6b7280;
}

@media (max-width: 768px) {
  .app-header {
    padding: 1.5rem 1rem;
  }

  .app-header h1 {
    font-size: 1.8rem;
  }

  .app-main {
    padding: 1rem;
  }

  .controls-section,
  .palette-section,
  .preview-section {
    padding: 1.1rem;
  }
}
</style>
