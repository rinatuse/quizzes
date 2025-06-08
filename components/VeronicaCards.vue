<template>
  <div class="container mx-auto px-4 py-8">
    <div class="text-center mb-8">
      <h1 class="text-4xl font-extrabold text-transparent bg-clip-text bg-gradient-to-r from-purple-600 to-indigo-600 mb-4">
        Карточки от Вероники
      </h1>
      <p class="text-gray-600 max-w-2xl mx-auto">Коллекция препаратов для изучения.</p>
    </div>

    <div class="flex flex-col md:flex-row justify-between items-center mb-6 gap-4">
      <div class="relative">
        <button
          @click="toggleDropdown"
          class="px-5 py-2 rounded-full bg-gray-100 text-gray-700 hover:bg-gray-200 flex items-center space-x-2"
        >
          <span>{{ selectedCategory ? selectedCategory : 'Все категории' }}</span>
          <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" :class="dropdownOpen ? 'transform rotate-180' : ''" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 9l-7 7-7-7" />
          </svg>
        </button>
        <div
          v-if="dropdownOpen"
          class="absolute z-10 mt-2 w-56 bg-white rounded-xl shadow-lg py-2 left-0 max-h-60 overflow-y-auto"
        >
          <button
            @click="selectCategory(null)"
            class="w-full text-left px-4 py-2 hover:bg-indigo-50 text-gray-700"
            :class="selectedCategory === null ? 'font-bold text-indigo-700 bg-indigo-50' : ''"
          >
            Все категории
          </button>
          <button
            v-for="cat in categories"
            :key="cat"
            @click="selectCategory(cat)"
            class="w-full text-left px-4 py-2 hover:bg-indigo-50 text-gray-700"
            :class="selectedCategory === cat ? 'font-bold text-indigo-700 bg-indigo-50' : ''"
          >
            {{ cat }}
          </button>
        </div>
      </div>

      <input
        v-model="searchQuery"
        type="text"
        placeholder="Поиск..."
        class="px-4 py-2 rounded-full border border-gray-300 focus:outline-none focus:ring-2 focus:ring-indigo-500 w-full md:w-60"
      />
    </div>

    <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
      <div
        v-for="med in filteredMeds"
        :key="med.id"
        class="bg-white rounded-xl shadow-lg p-6 hover:shadow-xl transition-all duration-300 cursor-pointer"
        @click="openDetails(med)"
      >
        <h3 class="text-xl font-bold text-indigo-700 mb-2">{{ med.title }}</h3>
        <p class="text-gray-600 mb-4">{{ med.category }}</p>
        <div class="text-sm text-gray-500 font-mono">{{ med.text }}</div>
      </div>
    </div>

    <div v-if="filteredMeds.length === 0" class="text-center text-gray-600 mt-6">
      Ничего не найдено.
    </div>

    <div
      v-if="selectedMed"
      class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50 p-4"
      @click.self="selectedMed = null"
    >
      <div
        class="bg-white rounded-xl shadow-2xl max-w-2xl w-full max-h-[90vh] overflow-y-auto p-8 relative"
        @click.stop
      >
        <button
          @click="selectedMed = null"
          class="absolute top-4 right-4 text-gray-500 hover:text-gray-700"
        >
          ✕
        </button>
        <h2 class="text-3xl font-bold text-indigo-700 mb-4">{{ selectedMed.title }}</h2>
        <div class="space-y-4">
          <div>
            <h3 class="font-semibold text-gray-700">Рецепт:</h3>
            <pre class="bg-gray-100 p-4 rounded-lg text-sm">{{ selectedMed.text }}</pre>
          </div>
          <div>
            <h3 class="font-semibold text-gray-700">Описание:</h3>
            <p class="text-gray-600">{{ selectedMed.description }}</p>
          </div>
          <div>
            <h3 class="font-semibold text-gray-700">Ключевые моменты:</h3>
            <ul class="list-disc list-inside text-gray-600">
              <li v-for="(point, idx) in selectedMed.keyPoints" :key="idx">{{ point }}</li>
            </ul>
          </div>
          <div>
            <h3 class="font-semibold text-gray-700">Категория:</h3>
            <span class="bg-indigo-100 text-indigo-700 px-3 py-1 rounded-full">{{ selectedMed.category }}</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onBeforeUnmount } from 'vue'
import { veronica } from '~/data/veronica.js'

const dropdownOpen = ref(false)
const selectedCategory = ref(null)
const searchQuery = ref('')
const selectedMed = ref(null)

const toggleDropdown = (event) => {
  event.stopPropagation()
  dropdownOpen.value = !dropdownOpen.value
}

const closeDropdownOnOutsideClick = () => {
  if (dropdownOpen.value) dropdownOpen.value = false
}

onMounted(() => {
  document.addEventListener('click', closeDropdownOnOutsideClick)
})

onBeforeUnmount(() => {
  document.removeEventListener('click', closeDropdownOnOutsideClick)
})

const categories = computed(() => [...new Set(veronica.map(m => m.category))])

const filteredMeds = computed(() => {
  let result = selectedCategory.value ? veronica.filter(m => m.category === selectedCategory.value) : veronica
  if (searchQuery.value) {
    const q = searchQuery.value.toLowerCase()
    result = result.filter(m =>
      m.title.toLowerCase().includes(q) ||
      m.text.toLowerCase().includes(q) ||
      m.category.toLowerCase().includes(q)
    )
  }
  return result
})

const selectCategory = (cat) => {
  selectedCategory.value = cat
  dropdownOpen.value = false
}

const openDetails = (med) => {
  selectedMed.value = med
}
</script>

<style scoped>
</style>
