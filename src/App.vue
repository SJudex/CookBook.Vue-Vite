<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'
import { recipes } from './data/recipes.js'

const selectedCategory = ref('Все')
const activeRecipe = ref(null)

// Директива для плавного появления карточек при скролле
const vReveal = {
  mounted(el) {
    el.classList.add('reveal-hidden')
    const observer = new IntersectionObserver(
      (entries) => {
        entries.forEach((entry) => {
          if (entry.isIntersecting) {
            entry.target.classList.add('reveal-visible')
            observer.unobserve(entry.target)
          }
        })
      },
      {
        threshold: 0.1,
        rootMargin: '0px 0px -25px 0px'
      }
    )
    observer.observe(el)
  }
}

// Собираем уникальные категории из массива рецептов
const categories = computed(() => {
  const unique = new Set(recipes.map((r) => r.category))
  return ['Все', ...unique]
})

// Фильтрация рецептов по выбранной категории
const filteredRecipes = computed(() => {
  if (selectedCategory.value === 'Все') return recipes
  return recipes.filter((r) => r.category === selectedCategory.value)
})

const openRecipe = (recipe) => {
  activeRecipe.value = recipe
  document.body.style.overflow = 'hidden'
}

const closeRecipe = () => {
  activeRecipe.value = null
  document.body.style.overflow = ''
}

// Закрытие модального окна по клавише Esc
const handleKeydown = (e) => {
  if (e.key === 'Escape' && activeRecipe.value) {
    closeRecipe()
  }
}

onMounted(() => window.addEventListener('keydown', handleKeydown))
onUnmounted(() => window.removeEventListener('keydown', handleKeydown))
</script>

<template>
  <div class="page">
    <!-- Шапка -->
    <header class="header">
      <div class="logo">Всегда вкусно</div>
      <p class="subtitle">любимые рецепты для <em>домашней кухни</em></p>
    </header>

    <!-- Фильтры -->
    <nav class="filters">
      <button
        v-for="cat in categories"
        :key="cat"
        :class="['filter-btn', { active: selectedCategory === cat }]"
        @click="selectedCategory = cat"
      >
        {{ cat }}
      </button>
    </nav>

    <!-- Сетка рецептов -->
    <main class="grid">
      <article
        v-for="(recipe, index) in filteredRecipes"
        :key="recipe.id"
        v-reveal
        :style="{ transitionDelay: `${(index % 4) * 75}ms` }"
        class="card"
        @click="openRecipe(recipe)"
      >
        <div class="card-img-wrap">
          <img :src="recipe.image" :alt="recipe.title" loading="lazy" />
        </div>
        <div class="card-meta">
          <span class="category">{{ recipe.category }}</span>
          <span class="dot">•</span>
          <span class="time">{{ recipe.time }}</span>
        </div>
        <h2 class="card-title">{{ recipe.title }}</h2>
      </article>
    </main>

    <!-- Модальное окно -->
    <Transition name="fade">
      <div v-if="activeRecipe" class="modal-backdrop" @click.self="closeRecipe">
        <div class="modal-content">
          <button class="close-btn" @click="closeRecipe" aria-label="Закрыть">✕</button>

          <div class="modal-hero">
            <img :src="activeRecipe.image" :alt="activeRecipe.title" />
          </div>

          <div class="modal-body">
            <div class="card-meta">
              <span class="category">{{ activeRecipe.category }}</span>
              <span class="dot">•</span>
              <span class="time">{{ activeRecipe.time }}</span>
            </div>
            <h1 class="modal-title">{{ activeRecipe.title }}</h1>
            <p class="modal-desc">{{ activeRecipe.description }}</p>

            <div class="recipe-details">
              <div class="ingredients">
                <h3>Ингредиенты</h3>
                <ul>
                  <li v-for="(item, idx) in activeRecipe.ingredients" :key="idx">
                    <label>
                      <input type="checkbox" />
                      <span>{{ item }}</span>
                    </label>
                  </li>
                </ul>
              </div>

              <div class="instructions">
                <h3>Приготовление</h3>
                <ol>
                  <li v-for="(step, idx) in activeRecipe.steps" :key="idx">
                    {{ step }}
                  </li>
                </ol>
              </div>
            </div>

            <!-- Блок комментариев от шефа -->
            <div v-if="activeRecipe.chefNotes?.length" class="chef-notes">
              <div class="chef-notes-header">
                <span class="chef-badge">Заметки от шефа</span>
              </div>
              <ul>
                <li v-for="(note, idx) in activeRecipe.chefNotes" :key="idx">
                  {{ note }}
                </li>
              </ul>
            </div>
          </div>
        </div>
      </div>
    </Transition>
  </div>
</template>
