<script setup>
const props = defineProps({
  colors: {
    type: Array,
    required: true,
  },
})

const emit = defineEmits(['toggle-lock', 'copy-color'])

function onToggleLock(index) {
  emit('toggle-lock', index)
}

function onCopy(index) {
  emit('copy-color', index)
}
</script>

<template>
  <div class="palette-bar">
    <div
      v-for="(item, index) in props.colors"
      :key="index"
      class="color-card"
      :style="{ backgroundColor: item.color }"
    >
      <div class="color-card-content">
        <div class="color-label" @click="onCopy(index)">
          <span>{{ item.label }}</span>
        </div>
        <div class="color-actions">
          <button
            class="icon-button"
            type="button"
            :title="item.locked ? 'Разблокировать' : 'Закрепить цвет'"
            @click="onToggleLock(index)"
          >
            <span v-if="item.locked">🔒</span>
            <span v-else>🔓</span>
          </button>
          <button
            class="icon-button"
            type="button"
            title="Копировать цвет"
            @click="onCopy(index)"
          >
            📋
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.palette-bar {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(120px, 1fr));
  gap: 1rem;
}

.color-card {
  position: relative;
  border-radius: 14px;
  min-height: 120px;
  overflow: hidden;
  box-shadow: 0 10px 25px rgba(15, 23, 42, 0.25);
  display: flex;
  align-items: flex-end;
  transition: transform 0.15s ease, box-shadow 0.15s ease;
}

.color-card:hover {
  transform: translateY(-4px);
  box-shadow: 0 16px 35px rgba(15, 23, 42, 0.35);
}

.color-card-content {
  width: 100%;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0.6rem 0.75rem;
  background: linear-gradient(
    to top,
    rgba(15, 23, 42, 0.85),
    rgba(15, 23, 42, 0.55),
    transparent
  );
  color: #f9fafb;
}

.color-label {
  font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, 'Liberation Mono',
    'Courier New', monospace;
  font-size: 0.85rem;
  cursor: pointer;
  user-select: none;
}

.color-actions {
  display: flex;
  align-items: center;
  gap: 0.25rem;
}

.icon-button {
  width: 28px;
  height: 28px;
  border-radius: 999px;
  border: none;
  background-color: rgba(15, 23, 42, 0.7);
  color: #e5e7eb;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  font-size: 0.8rem;
  transition: background-color 0.15s ease, transform 0.1s ease;
}

.icon-button:hover {
  background-color: rgba(15, 23, 42, 0.9);
  transform: translateY(-1px);
}

.icon-button:active {
  transform: translateY(0);
}
</style>


