<template>
    <div class="min-h-screen bg-gradient-to-br from-gray-50 to-gray-100">
      <div class="container mx-auto px-3 sm:px-4 py-4 sm:py-8">
        <!-- Заголовок -->
        <div class="text-center mb-6 sm:mb-10">
          <h1 class="text-2xl sm:text-4xl font-extrabold text-transparent bg-clip-text bg-gradient-to-r from-green-600 to-teal-600 mb-2 sm:mb-4">
            Экзаменационные билеты
          </h1>
          <p class="text-sm sm:text-base text-gray-600 max-w-2xl mx-auto px-4">
            Подготовка к экзамену по врачебной рецептуре. Кликните на карточку для просмотра ответа.
          </p>
        </div>
  
        <!-- Прогресс-бар -->
        <div v-if="mode === 'detail'" class="mb-6 px-4 sm:px-8">
          <div class="flex justify-between text-sm text-gray-600 mb-2">
            <span>Прогресс по билету {{ selectedTicket }}</span>
            <span>{{ revealedCount }} из {{ currentTicketLength }}</span>
          </div>
          <div class="w-full bg-gray-200 rounded-full h-2">
            <div
              class="bg-gradient-to-r from-green-500 to-teal-500 h-2 rounded-full transition-all duration-300"
              :style="{ width: progressPercentage + '%' }"
            ></div>
          </div>
        </div>
  
        <!-- Режимы отображения -->
        <div class="flex justify-center mb-6">
          <div class="inline-flex bg-white rounded-full shadow-md">
            <button
              @click="switchMode('list')"
              class="px-4 sm:px-5 py-2 sm:py-3 rounded-full text-xs sm:text-sm font-semibold transition-all duration-300"
              :class="mode === 'list' 
                ? 'bg-gradient-to-r from-green-600 to-teal-600 text-white transform scale-105' 
                : 'text-gray-700 hover:bg-gray-100'"
            >
              Все рецепты
            </button>
            <button
              @click="switchMode('detail')"
              class="px-4 sm:px-5 py-2 sm:py-3 rounded-full text-xs sm:text-sm font-semibold transition-all duration-300"
              :class="mode === 'detail' 
                ? 'bg-gradient-to-r from-green-600 to-teal-600 text-white transform scale-105' 
                : 'text-gray-700 hover:bg-gray-100'"
            >
              По билетам
            </button>
          </div>
        </div>
  
        <!-- Поиск и фильтры -->
        <div v-if="mode === 'list'" class="mb-6 px-4">
          <div class="max-w-md mx-auto relative">
            <input
              v-model="searchQuery"
              type="text"
              placeholder="Поиск по рецептам..."
              class="w-full pl-10 pr-4 py-3 rounded-xl border border-gray-300 focus:ring-2 focus:ring-green-500 focus:border-transparent"
            />
            <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 absolute left-3 top-3.5 text-gray-400" fill="none" viewBox="0 0 24 24" stroke="currentColor">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z" />
            </svg>
          </div>
        </div>
  
        <transition name="fade" mode="out-in">
          <!-- Режим списка всех рецептов -->
          <div v-if="mode === 'list'" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4 sm:gap-6 px-4">
            <div
              v-for="prescription in filteredPrescriptions"
              :key="prescription.id"
              class="bg-white rounded-xl shadow-lg overflow-hidden hover:shadow-xl transition-all cursor-pointer group"
              @click="showDetails(prescription)"
            >
              <div class="p-4 sm:p-6">
                <div class="flex justify-between items-start mb-3">
                  <h3 class="text-base sm:text-lg font-bold text-gray-800 flex-1 group-hover:text-green-600 transition-colors">
                    {{ prescription.title }}
                  </h3>
                  <span class="bg-green-100 text-green-800 text-xs px-2 py-1 rounded-full ml-2 flex-shrink-0">
                    Билет {{ prescription.ticketNumber }}
                  </span>
                </div>
  
                <p class="text-sm text-gray-600 mb-3 line-clamp-2">{{ prescription.description }}</p>
  
                <div class="flex justify-between items-center">
                  <span class="text-xs text-gray-500">{{ prescription.category }}</span>
                  <button class="text-green-600 hover:text-green-700 font-medium text-sm flex items-center gap-1">
                    Подробнее
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5l7 7-7 7" />
                    </svg>
                  </button>
                </div>
              </div>
            </div>
          </div>
  
          <!-- Режим отображения по билетам -->
          <div v-else-if="mode === 'detail'" class="space-y-6">
            <!-- Выбор билета -->
            <div class="flex justify-center mb-8 px-4">
              <div class="bg-white rounded-lg shadow-md p-1 w-full sm:w-auto overflow-x-auto">
                <div class="flex gap-1 sm:gap-2 min-w-max">
                  <button
                    v-for="ticket in examTickets"
                    :key="ticket.ticketNumber"
                    @click="selectTicket(ticket.ticketNumber)"
                    class="px-3 sm:px-4 py-2 rounded-lg text-xs sm:text-sm font-medium transition-all whitespace-nowrap"
                    :class="selectedTicket === ticket.ticketNumber 
                      ? 'bg-gradient-to-r from-green-600 to-teal-600 text-white transform scale-105' 
                      : 'text-gray-700 hover:bg-gray-100'"
                  >
                    Билет {{ ticket.ticketNumber }}
                  </button>
                </div>
              </div>
            </div>
  
            <!-- Рецепты выбранного билета -->
            <div v-if="selectedTicketData" class="px-4">
              <div class="bg-white rounded-xl shadow-lg p-4 sm:p-8">
                <div class="flex justify-between items-center mb-6">
                  <h2 class="text-xl sm:text-2xl font-bold text-gray-800">
                    Билет № {{ selectedTicketData.ticketNumber }}
                  </h2>
                  <button
                    @click="resetTicket"
                    class="text-sm text-gray-600 hover:text-gray-800 flex items-center gap-1"
                  >
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 4v5h.582m15.356 2A8.001 8.001 0 004.582 9m0 0H9m11 11v-5h-.581m0 0a8.003 8.003 0 01-15.357-2m15.357 2H15" />
                    </svg>
                    Сбросить
                  </button>
                </div>
  
                <div class="space-y-4 sm:space-y-6">
                  <!-- Карточки рецептов -->
                  <div
                    v-for="(prescription, index) in selectedTicketData.prescriptions"
                    :key="prescription.id"
                    class="border rounded-xl overflow-hidden transition-all duration-300"
                    :class="revealedPrescriptions[prescription.id] ? 'border-green-500 shadow-md' : 'border-gray-200'"
                  >
                    <div
                      @click="togglePrescription(prescription.id)"
                      class="p-4 cursor-pointer hover:bg-gray-50 transition-colors"
                    >
                      <div class="flex items-start">
                        <div class="flex-shrink-0 w-8 h-8 bg-gradient-to-r from-green-500 to-teal-500 rounded-full flex items-center justify-center mr-3">
                          <span class="text-white font-bold text-sm">{{ index + 1 }}</span>
                        </div>
                        <div class="flex-1">
                          <h3 class="text-base sm:text-lg font-bold text-gray-800 mb-2">
                            {{ prescription.title }}
                          </h3>
                          <div class="flex flex-wrap items-center gap-2 text-xs sm:text-sm text-gray-500">
                            <span class="bg-gray-100 px-2 py-1 rounded">{{ prescription.category }}</span>
                            <span class="flex items-center gap-1">
                              <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 12a3 3 0 11-6 0 3 3 0 016 0z" />
                                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M2.458 12C3.732 7.943 7.523 5 12 5c4.478 0 8.268 2.943 9.542 7-1.274 4.057-5.064 7-9.542 7-4.477 0-8.268-2.943-9.542-7z" />
                              </svg>
                              Кликните для просмотра
                            </span>
                          </div>
                        </div>
                        <div class="flex-shrink-0 ml-2">
                          <svg
                            xmlns="http://www.w3.org/2000/svg"
                            class="h-5 w-5 transition-transform duration-300"
                            :class="revealedPrescriptions[prescription.id] ? 'rotate-180 text-green-500' : 'text-gray-400'"
                            fill="none"
                            viewBox="0 0 24 24"
                            stroke="currentColor"
                          >
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 9l-7 7-7-7" />
                          </svg>
                        </div>
                      </div>
                    </div>
  
                    <!-- Содержимое рецепта (скрыто/показано) -->
                    <transition
                      enter-active-class="transition-all duration-300 ease-out"
                      enter-from-class="transform opacity-0 max-h-0"
                      enter-to-class="transform opacity-100 max-h-screen"
                      leave-active-class="transition-all duration-300 ease-in"
                      leave-from-class="transform opacity-100 max-h-screen"
                      leave-to-class="transform opacity-0 max-h-0"
                    >
                      <div v-if="revealedPrescriptions[prescription.id]" class="border-t border-gray-200">
                        <div class="p-4 space-y-4">
                          <!-- Рецепт -->
                          <div>
                            <h4 class="font-semibold text-gray-700 mb-2 text-sm sm:text-base">Рецепт:</h4>
                            <div class="bg-gray-50 p-3 sm:p-4 rounded-lg relative">
                              <pre class="whitespace-pre-wrap font-mono text-xs sm:text-sm text-gray-700">{{ prescription.text }}</pre>
                              <button
                                @click.stop="copyToClipboard(prescription.text)"
                                class="absolute top-2 right-2 p-2 rounded bg-white shadow-sm hover:shadow-md transition-shadow"
                                title="Копировать рецепт"
                              >
                                <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4 text-gray-600" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 16H6a2 2 0 01-2-2V6a2 2 0 012-2h8a2 2 0 012 2v2m-6 12h8a2 2 0 002-2v-8a2 2 0 00-2-2h-8a2 2 0 00-2 2v8a2 2 0 002 2z" />
                                </svg>
                              </button>
                            </div>
                          </div>
  
                          <!-- Описание -->
                          <div>
                            <h4 class="font-semibold text-gray-700 mb-2 text-sm sm:text-base">Описание:</h4>
                            <p class="text-sm sm:text-base text-gray-600">{{ prescription.description }}</p>
                          </div>
  
                          <!-- Ключевые моменты -->
                          <div>
                            <h4 class="font-semibold text-gray-700 mb-2 text-sm sm:text-base">Ключевые моменты:</h4>
                            <ul class="list-disc list-inside text-sm sm:text-base text-gray-600 space-y-1">
                              <li v-for="point in prescription.keyPoints" :key="point">{{ point }}</li>
                            </ul>
                          </div>
  
                          <!-- Статус изучения -->
                          <div class="flex items-center justify-between pt-4 border-t border-gray-100">
                            <button
                              @click.stop="toggleLearned(prescription.id)"
                              class="flex items-center gap-2 text-sm font-medium transition-colors"
                              :class="learnedPrescriptions[prescription.id] ? 'text-green-600' : 'text-gray-500'"
                            >
                              <svg
                                xmlns="http://www.w3.org/2000/svg"
                                class="h-5 w-5"
                                :class="learnedPrescriptions[prescription.id] ? 'text-green-600' : 'text-gray-400'"
                                fill="none"
                                viewBox="0 0 24 24"
                                stroke="currentColor"
                              >
                                <path
                                  stroke-linecap="round"
                                  stroke-linejoin="round"
                                  stroke-width="2"
                                  d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z"
                                />
                              </svg>
                              {{ learnedPrescriptions[prescription.id] ? 'Изучено' : 'Отметить как изученное' }}
                            </button>
                          </div>
                        </div>
                      </div>
                    </transition>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </transition>
  
        <!-- Уведомление о копировании -->
        <transition
          enter-active-class="transition-all duration-300 ease-out"
          enter-from-class="transform translate-y-full opacity-0"
          enter-to-class="transform translate-y-0 opacity-100"
          leave-active-class="transition-all duration-300 ease-in"
          leave-from-class="transform translate-y-0 opacity-100"
          leave-to-class="transform translate-y-full opacity-0"
        >
          <div
            v-if="showCopyNotification"
            class="fixed bottom-4 right-4 bg-gray-900 text-white px-4 py-2 rounded-lg shadow-lg z-50"
          >
            Рецепт скопирован!
          </div>
        </transition>
  
        <!-- Модальное окно с деталями -->
        <div
          v-if="selectedPrescription"
          class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-40 p-4"
          @click.self="selectedPrescription = null"
        >
          <div
            class="bg-white rounded-xl shadow-2xl max-w-3xl w-full max-h-[90vh] overflow-y-auto relative"
            @click.stop
          >
            <div class="sticky top-0 bg-white border-b border-gray-200 p-4 sm:p-6 z-10">
              <button
                @click="selectedPrescription = null"
                class="absolute top-4 right-4 p-2 rounded-full hover:bg-gray-100 transition-colors"
              >
                <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6 text-gray-500" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
                </svg>
              </button>
  
              <h2 class="text-xl sm:text-2xl font-bold text-green-700 pr-8">
                {{ selectedPrescription.title }}
              </h2>
  
              <div class="flex flex-wrap items-center gap-2 mt-3">
                <span class="bg-green-100 text-green-800 text-sm px-3 py-1 rounded-full">
                  Билет {{ selectedPrescription.ticketNumber }}
                </span>
                <span class="bg-gray-100 text-gray-800 text-sm px-3 py-1 rounded-full">
                  {{ selectedPrescription.category }}
                </span>
              </div>
            </div>
  
            <div class="p-4 sm:p-6 space-y-6">
              <div>
                <h3 class="font-semibold text-gray-700 mb-2">Рецепт:</h3>
                <div class="bg-gray-50 p-4 rounded-lg relative">
                  <pre class="whitespace-pre-wrap font-mono text-sm">{{ selectedPrescription.text }}</pre>
                  <button
                    @click="copyToClipboard(selectedPrescription.text)"
                    class="absolute top-2 right-2 p-2 rounded bg-white shadow-sm hover:shadow-md transition-shadow"
                    title="Копировать рецепт"
                  >
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4 text-gray-600" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 16H6a2 2 0 01-2-2V6a2 2 0 012-2h8a2 2 0 012 2v2m-6 12h8a2 2 0 002-2v-8a2 2 0 00-2-2h-8a2 2 0 00-2 2v8a2 2 0 002 2z" />
                    </svg>
                  </button>
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
    </div>
  </template>
  
  <script setup>
  import { ref, computed, onMounted } from 'vue'
  import { examTickets } from '~/data/pulmonary.js'
  
  // Состояние
  const mode = ref('detail')
  const selectedTicket = ref(1)
  const selectedPrescription = ref(null)
  const searchQuery = ref('')
  const revealedPrescriptions = ref({})
  const learnedPrescriptions = ref({})
  const showCopyNotification = ref(false)
  
  // Загрузка сохраненного прогресса
  onMounted(() => {
    const savedProgress = localStorage.getItem('examProgress')
    if (savedProgress) {
      const data = JSON.parse(savedProgress)
      revealedPrescriptions.value = data.revealed || {}
      learnedPrescriptions.value = data.learned || {}
    }
  })
  
  // Сохранение прогресса
  const saveProgress = () => {
    localStorage.setItem('examProgress', JSON.stringify({
      revealed: revealedPrescriptions.value,
      learned: learnedPrescriptions.value
    }))
  }
  
  // Вычисляемые свойства
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
  
  const filteredPrescriptions = computed(() => {
    if (!searchQuery.value) return allPrescriptions.value
    
    const query = searchQuery.value.toLowerCase()
    return allPrescriptions.value.filter(p => 
      p.title.toLowerCase().includes(query) ||
      p.description.toLowerCase().includes(query) ||
      p.category.toLowerCase().includes(query)
    )
  })
  
  const selectedTicketData = computed(() => {
    return examTickets.find(ticket => ticket.ticketNumber === selectedTicket.value)
  })
  
  const currentTicketLength = computed(() => {
    return selectedTicketData.value?.prescriptions.length || 0
  })
  
  const revealedCount = computed(() => {
    if (!selectedTicketData.value) return 0
    return selectedTicketData.value.prescriptions.filter(p => revealedPrescriptions.value[p.id]).length
  })
  
  const progressPercentage = computed(() => {
    if (currentTicketLength.value === 0) return 0
    return (revealedCount.value / currentTicketLength.value) * 100
  })
  
  // Методы
  const switchMode = (newMode) => {
    mode.value = newMode
    searchQuery.value = ''
  }
  
  const selectTicket = (ticketNumber) => {
    selectedTicket.value = ticketNumber
  }
  
  const showDetails = (prescription) => {
    selectedPrescription.value = prescription
  }
  
  const togglePrescription = (id) => {
    revealedPrescriptions.value[id] = !revealedPrescriptions.value[id]
    saveProgress()
  }
  
  const toggleLearned = (id) => {
    learnedPrescriptions.value[id] = !learnedPrescriptions.value[id]
    saveProgress()
  }
  
  const resetTicket = () => {
    if (!selectedTicketData.value) return
    
    selectedTicketData.value.prescriptions.forEach(p => {
      revealedPrescriptions.value[p.id] = false
      learnedPrescriptions.value[p.id] = false
    })
    saveProgress()
  }
  
  const copyToClipboard = async (text) => {
    try {
      await navigator.clipboard.writeText(text)
      showCopyNotification.value = true
      setTimeout(() => {
        showCopyNotification.value = false
      }, 2000)
    } catch (err) {
      console.error('Failed to copy:', err)
    }
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
  
  .line-clamp-2 {
    overflow: hidden;
    display: -webkit-box;
    -webkit-line-clamp: 2;
    -webkit-box-orient: vertical;
  }
  
  /* Кастомная полоса прокрутки для мобильных */
  .overflow-x-auto::-webkit-scrollbar {
    height: 4px;
  }
  
  .overflow-x-auto::-webkit-scrollbar-track {
    background: #f1f1f1;
    border-radius: 2px;
  }
  
  .overflow-x-auto::-webkit-scrollbar-thumb {
    background: #cbd5e0;
    border-radius: 2px;
  }
  
  .overflow-x-auto::-webkit-scrollbar-thumb:hover {
    background: #a0aec0;
  }
  </style>