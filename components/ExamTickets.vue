// components/ExamTickets.vue

<template>
  <div class="container mx-auto px-4 py-8">
    <!-- Заголовок и описание -->
    <div class="text-center mb-8">
      <h1 class="text-4xl font-extrabold text-transparent bg-clip-text bg-gradient-to-r from-green-600 to-teal-600 mb-4">
        Экзаменационные билеты
      </h1>
      <p class="text-gray-600 max-w-2xl mx-auto">
        Подготовка к экзамену по врачебной рецептуре
      </p>
    </div>

    <!-- Режимы отображения -->
    <div class="flex justify-center mb-6">
      <div class="inline-flex bg-white rounded-full shadow-md">
        <button 
          @click="mode = 'list'" 
          class="px-5 py-3 rounded-full text-sm font-semibold transition-colors"
          :class="mode === 'list' ? 'bg-gradient-to-r from-green-600 to-teal-600 text-white' : 'text-gray-700 hover:bg-gray-100'"
        >
          Все билеты
        </button>
        <button 
          @click="mode = 'detail'" 
          class="px-5 py-3 rounded-full text-sm font-semibold transition-colors"
          :class="mode === 'detail' ? 'bg-gradient-to-r from-green-600 to-teal-600 text-white' : 'text-gray-700 hover:bg-gray-100'"
        >
          По билетам
        </button>
      </div>
    </div>

    <transition name="fade" mode="out-in">
      <!-- Режим списка всех рецептов -->
      <div v-if="mode === 'list'" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
        <div v-for="prescription in allPrescriptions" :key="prescription.id" 
          class="bg-white rounded-xl shadow-lg p-6 hover:shadow-xl transition-all cursor-pointer"
          @click="showDetails(prescription)"
        >
          <div class="flex justify-between items-start mb-3">
            <h3 class="text-lg font-bold text-gray-800 flex-1">{{ prescription.title }}</h3>
            <span class="bg-green-100 text-green-800 text-xs px-2 py-1 rounded-full ml-2">
              Билет {{ prescription.ticketNumber }}
            </span>
          </div>
          
          <p class="text-gray-600 text-sm mb-3">{{ prescription.description }}</p>
          
          <div class="bg-gray-50 p-3 rounded-lg font-mono text-sm">
            <pre class="whitespace-pre-wrap">{{ prescription.text }}</pre>
          </div>
          
          <div class="mt-3 flex justify-between items-center">
            <span class="text-xs text-gray-500">{{ prescription.category }}</span>
            <button class="text-green-600 hover:text-green-700 font-medium text-sm">
              Подробнее →
            </button>
          </div>
        </div>
      </div>

      <!-- Режим отображения по билетам -->
      <div v-else-if="mode === 'detail'" class="space-y-6">
        <!-- Выбор билета -->
        <div class="flex justify-center mb-8">
          <div class="inline-flex bg-white rounded-lg shadow-md p-1 flex-wrap gap-2">
            <button 
              v-for="ticket in examTickets" 
              :key="ticket.ticketNumber"
              @click="selectedTicket = ticket.ticketNumber"
              class="px-4 py-2 rounded-lg text-sm font-medium transition-all"
              :class="selectedTicket === ticket.ticketNumber 
                ? 'bg-gradient-to-r from-green-600 to-teal-600 text-white' 
                : 'text-gray-700 hover:bg-gray-100'"
            >
              Билет {{ ticket.ticketNumber }}
            </button>
          </div>
        </div>

        <!-- Рецепты выбранного билета -->
        <div v-if="selectedTicketData" class="bg-white rounded-xl shadow-lg p-8">
          <h2 class="text-2xl font-bold text-gray-800 mb-6">
            Билет № {{ selectedTicketData.ticketNumber }}
          </h2>
          
          <div class="space-y-6">
            <div 
              v-for="(prescription, index) in selectedTicketData.prescriptions" 
              :key="prescription.id"
              class="border-b border-gray-200 pb-6 last:border-b-0"
            >
              <div class="flex items-start">
                <div class="flex-shrink-0 w-8 h-8 bg-green-100 rounded-full flex items-center justify-center mr-4">
                  <span class="text-green-700 font-bold text-sm">{{ index + 1 }}</span>
                </div>
                
                <div class="flex-1">
                  <h3 class="text-xl font-bold text-gray-800 mb-3">{{ prescription.title }}</h3>
                  
                  <div class="bg-gray-50 p-4 rounded-lg mb-4">
                    <pre class="whitespace-pre-wrap font-mono text-sm text-gray-700">{{ prescription.text }}</pre>
                  </div>
                  
                  <div class="text-gray-600 mb-4">{{ prescription.description }}</div>
                  
                  <div class="space-y-2">
                    <h4 class="font-semibold text-gray-700">Ключевые моменты:</h4>
                    <ul class="list-disc list-inside text-gray-600 space-y-1">
                      <li v-for="point in prescription.keyPoints" :key="point">{{ point }}</li>
                    </ul>
                  </div>
                  
                  <div class="mt-4">
                    <span class="inline-block bg-green-100 text-green-800 text-xs px-3 py-1 rounded-full">
                      {{ prescription.category }}
                    </span>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </transition>

    <!-- Модальное окно с деталями -->
    <div 
      v-if="selectedPrescription" 
      class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50 p-4"
      @click.self="selectedPrescription = null"
    >
      <div 
        class="bg-white rounded-xl shadow-2xl max-w-3xl w-full max-h-[90vh] overflow-y-auto p-8 relative"
        @click.stop
      >
        <button 
          @click="selectedPrescription = null"
          class="absolute top-4 right-4 text-gray-500 hover:text-gray-700"
        >
          ✕
        </button>
        
        <h2 class="text-2xl font-bold text-green-700 mb-4">
          {{ selectedPrescription.title }}
        </h2>
        
        <div class="flex items-center gap-4 mb-6">
          <span class="bg-green-100 text-green-800 text-sm px-3 py-1 rounded-full">
            Билет {{ selectedPrescription.ticketNumber }}
          </span>
          <span class="bg-gray-100 text-gray-800 text-sm px-3 py-1 rounded-full">
            {{ selectedPrescription.category }}
          </span>
        </div>
        
        <div class="space-y-6">
          <div>
            <h3 class="font-semibold text-gray-700 mb-2">Рецепт:</h3>
            <div class="bg-gray-50 p-4 rounded-lg">
              <pre class="whitespace-pre-wrap font-mono text-sm">{{ selectedPrescription.text }}</pre>
            </div>
          </div>
          
          <div>
            <h3 class="font-semibold text-gray-700 mb-2">Описание:</h3>
            <p class="text-gray-600">{{ selectedPrescription.description }}</p>
          </div>
          
          <div>
            <h3 class="font-semibold text-gray-700 mb-2">Ключевые моменты:</h3>
            <ul class="list-disc list-inside text-gray-600 space-y-2">
              <li v-for="(point, index) in selectedPrescription.keyPoints" :key="index">
                {{ point }}
              </li>
            </ul>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import { examTickets } from '~/data/pulmonary.js'

// Режим отображения
const mode = ref('detail')
const selectedTicket = ref(1)
const selectedPrescription = ref(null)

// Получение всех рецептов для режима списка
const allPrescriptions = computed(() => {
  const prescriptions = []
  examTickets.forEach(ticket => {
    ticket.prescriptions.forEach(prescription => {
      prescriptions.push({
        ...prescription,
        ticketNumber: ticket.ticketNumber
      })
    })
  })
  return prescriptions
})

// Получение данных выбранного билета
const selectedTicketData = computed(() => {
  return examTickets.find(ticket => ticket.ticketNumber === selectedTicket.value)
})

// Показать детали рецепта
const showDetails = (prescription) => {
  selectedPrescription.value = prescription
}
</script>

<style scoped>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>