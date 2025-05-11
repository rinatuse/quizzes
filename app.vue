// app.vue

<template>
  <div class="container mx-auto py-8 px-4">
    <h1 class="text-3xl font-bold text-center mb-8 text-indigo-700">Медицинский справочник</h1>
    
    <!-- Навигация между компонентами -->
    <div class="flex justify-center mb-8">
      <div class="bg-white p-1 rounded-full shadow-md inline-flex">
        <button 
          @click="activeComponent = 'pharma'"
          class="px-6 py-2.5 rounded-full text-sm font-medium transition-all duration-300 transform"
          :class="activeComponent === 'pharma' ? 'bg-gradient-to-r from-purple-600 to-indigo-600 text-white scale-105 shadow-lg' : 'text-gray-700 hover:bg-gray-100'"
        >
          Фармакология
        </button>
        <button 
          @click="activeComponent = 'emergency'"
          class="px-6 py-2.5 rounded-full text-sm font-medium transition-all duration-300 transform"
          :class="activeComponent === 'emergency' ? 'bg-gradient-to-r from-red-600 to-orange-600 text-white scale-105 shadow-lg' : 'text-gray-700 hover:bg-gray-100'"
        >
          Экстренные рецепты
        </button>
        <button 
          @click="activeComponent = 'exam'"
          class="px-6 py-2.5 rounded-full text-sm font-medium transition-all duration-300 transform"
          :class="activeComponent === 'exam' ? 'bg-gradient-to-r from-green-600 to-teal-600 text-white scale-105 shadow-lg' : 'text-gray-700 hover:bg-gray-100'"
        >
          Экзамен
        </button>
      </div>
    </div>
    
    <!-- Отображение активного компонента -->
    <transition name="fade" mode="out-in">
      <component :is="currentComponent" />
    </transition>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import PharmaCards from '~/components/PharmaCards.vue'
import EmergencyCards from '~/components/EmergencyCards.vue'
import ExamTickets from '~/components/ExamTickets.vue'

// Активный компонент
const activeComponent = ref('pharma')

// Компьютед свойство для определения текущего компонента
const currentComponent = computed(() => {
  switch (activeComponent.value) {
    case 'pharma':
      return PharmaCards
    case 'emergency':
      return EmergencyCards
    case 'exam':
      return ExamTickets
    default:
      return PharmaCards
  }
})
</script>

<style>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

body {
  font-family: 'Montserrat', sans-serif;
  background: linear-gradient(135deg, #f4f4f7 0%, #e9ecef 100%);
  min-height: 100vh;
  color: #1f2937;
  line-height: 1.6;
}

.container {
  max-width: 1200px;
}
</style>