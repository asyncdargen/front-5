<script setup>
import { computed } from 'vue'

const props = defineProps({
  palette: {
    type: Array,
    required: true,
  },
  dark: {
    type: Boolean,
    default: false,
  },
})

const primaryColor = computed(() => props.palette[0]?.color || '#4F46E5')
const accentColor = computed(() => props.palette[1]?.color || '#EC4899')
const backgroundColor = computed(() => (props.dark ? '#020617' : '#F9FAFB'))
const cardColor = computed(() => (props.dark ? '#020617' : '#FFFFFF'))
</script>

<template>
  <div class="mockup" :style="{ backgroundColor: backgroundColor }">
    <div class="mockup-inner">
      <header class="mockup-header" :style="{ borderBottomColor: primaryColor }">
        <div class="logo" :style="{ backgroundColor: primaryColor }"></div>
        <nav class="nav">
          <span class="nav-item">Главная</span>
          <span class="nav-item">Палитры</span>
          <span class="nav-item">Избранное</span>
        </nav>
        <button class="nav-button" type="button" :style="{ backgroundColor: accentColor }">
          Создать палитру
        </button>
      </header>

      <main class="mockup-main">
        <section class="hero">
          <div class="hero-text">
            <h3 :style="{ color: primaryColor }">Ваша новая палитра</h3>
            <p>
              Используйте цвета ниже для оформления кнопок, карточек, фона и акцентных элементов
              интерфейса.
            </p>
          </div>
          <div class="hero-badge" :style="{ borderColor: accentColor, color: accentColor }">
            WCAG preview
          </div>
        </section>

        <section class="mockup-grid">
          <article class="mockup-card" :style="{ backgroundColor: cardColor }">
            <div class="card-header">
              <span class="card-tag" :style="{ backgroundColor: primaryColor }">Кнопка</span>
              <button
                class="card-button"
                type="button"
                :style="{ backgroundColor: primaryColor }"
              >
                Действие
              </button>
            </div>
            <p>
              Пример использования основного цвета для кнопок. Следите за достаточным контрастом с
              текстом.
            </p>
          </article>

          <article class="mockup-card" :style="{ backgroundColor: cardColor }">
            <div class="card-header">
              <span class="card-tag" :style="{ backgroundColor: accentColor }">Акцент</span>
              <button
                class="card-button ghost"
                type="button"
                :style="{ color: accentColor, borderColor: accentColor }"
              >
                Подробнее
              </button>
            </div>
            <p>
              Акцентный цвет хорошо подходит для ссылок, вторичных кнопок и важных элементов
              интерфейса.
            </p>
          </article>

          <article class="mockup-card wide" :style="{ backgroundColor: cardColor }">
            <h4>Список элементов</h4>
            <ul class="list">
              <li v-for="(item, index) in palette" :key="index">
                <span class="list-dot" :style="{ backgroundColor: item.color }"></span>
                <span>Элемент интерфейса с использованием цвета {{ index + 1 }}</span>
              </li>
            </ul>
          </article>
        </section>
      </main>
    </div>
  </div>
</template>

<style scoped>
.mockup {
  border-radius: 18px;
  padding: 1rem;
  border: 1px solid rgba(148, 163, 184, 0.5);
}

.mockup-inner {
  border-radius: 14px;
  overflow: hidden;
  box-shadow: 0 12px 35px rgba(15, 23, 42, 0.25);
}

.mockup-header {
  display: flex;
  align-items: center;
  gap: 1rem;
  padding: 0.9rem 1.2rem;
  border-bottom-width: 3px;
  border-bottom-style: solid;
  background: linear-gradient(90deg, rgba(15, 23, 42, 0.95), rgba(15, 23, 42, 0.85));
}

.logo {
  width: 28px;
  height: 28px;
  border-radius: 999px;
}

.nav {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  color: #e5e7eb;
  font-size: 0.85rem;
}

.nav-item {
  opacity: 0.8;
}

.nav-button {
  margin-left: auto;
  padding: 0.4rem 0.9rem;
  border-radius: 999px;
  border: none;
  color: #0f172a;
  font-size: 0.8rem;
  font-weight: 600;
  cursor: pointer;
}

.mockup-main {
  padding: 1rem 1.25rem 1.2rem;
}

.hero {
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  gap: 1rem;
  margin-bottom: 1.2rem;
}

.hero-text h3 {
  font-size: 1.05rem;
  margin-bottom: 0.25rem;
}

.hero-text p {
  font-size: 0.85rem;
  color: #4b5563;
}

.hero-badge {
  align-self: flex-start;
  font-size: 0.75rem;
  padding: 0.3rem 0.6rem;
  border-radius: 999px;
  border-width: 1px;
  border-style: solid;
}

.mockup-grid {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 0.9rem;
}

.mockup-card {
  border-radius: 12px;
  padding: 0.9rem;
  border: 1px solid rgba(148, 163, 184, 0.45);
}

.mockup-card.wide {
  grid-column: span 2;
}

.mockup-card h4 {
  font-size: 0.95rem;
  margin-bottom: 0.4rem;
}

.mockup-card p {
  font-size: 0.8rem;
  color: #4b5563;
}

.card-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 0.5rem;
  margin-bottom: 0.4rem;
}

.card-tag {
  font-size: 0.75rem;
  padding: 0.15rem 0.5rem;
  border-radius: 999px;
  color: #f9fafb;
}

.card-button {
  padding: 0.25rem 0.75rem;
  border-radius: 999px;
  border: none;
  font-size: 0.75rem;
  font-weight: 600;
  cursor: pointer;
  color: #f9fafb;
}

.card-button.ghost {
  background-color: transparent;
  border-width: 1px;
  border-style: solid;
}

.list {
  list-style: none;
  margin: 0;
  padding: 0;
  display: flex;
  flex-direction: column;
  gap: 0.3rem;
}

.list li {
  display: flex;
  align-items: center;
  gap: 0.4rem;
  font-size: 0.8rem;
  color: #4b5563;
}

.list-dot {
  width: 12px;
  height: 12px;
  border-radius: 999px;
  border: 1px solid rgba(15, 23, 42, 0.3);
}

@media (max-width: 768px) {
  .mockup-main {
    padding: 0.75rem 0.9rem 0.9rem;
  }

  .mockup-grid {
    grid-template-columns: 1fr;
  }

  .mockup-card.wide {
    grid-column: span 1;
  }

  .hero {
    flex-direction: column;
  }
}
</style>


