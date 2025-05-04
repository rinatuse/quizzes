<template>
  <div class="container mx-auto px-4 py-8">
    <!-- Заголовок и описание -->
    <div class="text-center mb-8">
      <h1 class="text-4xl font-extrabold text-transparent bg-clip-text bg-gradient-to-r from-red-600 to-orange-500 mb-4">
        Экстренные медикаменты
      </h1>
      <p class="text-gray-600 max-w-2xl mx-auto">
        Интерактивные карточки для изучения лекарств, применяемых в скорой медицинской помощи
      </p>
    </div>

    <!-- Режимы просмотра -->
    <div class="flex justify-center mb-6">
      <div class="inline-flex bg-white rounded-full shadow-md">
        <button 
          @click="mode = 'cards'" 
          class="px-5 py-3 rounded-full text-sm font-semibold transition-colors"
          :class="mode === 'cards' ? 'bg-gradient-to-r from-red-600 to-orange-500 text-white' : 'text-gray-700 hover:bg-gray-100'"
        >
          Карточки
        </button>
        <button 
          @click="mode = 'quiz'" 
          class="px-5 py-3 rounded-full text-sm font-semibold transition-colors"
          :class="mode === 'quiz' ? 'bg-gradient-to-r from-red-600 to-orange-500 text-white' : 'text-gray-700 hover:bg-gray-100'"
        >
          Тестирование
        </button>
      </div>
    </div>
    
    <!-- Панель с фильтрами -->
    <div class="flex flex-col md:flex-row justify-center items-center gap-4 mb-6">
      <!-- Фильтры по категориям -->
      <div class="relative w-full max-w-xs">
        <button 
          @click="toggleCategoryDropdown" 
          class="w-full flex items-center justify-between px-4 py-3 bg-white rounded-xl shadow-md hover:shadow-lg transition-shadow"
        >
          <span>{{ selectedCategory ? selectedCategory : 'Все категории' }}</span>
          <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 text-gray-400" :class="dropdownOpen ? 'transform rotate-180' : ''" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 9l-7 7-7-7" />
          </svg>
        </button>
        
        <div v-if="dropdownOpen" class="absolute z-10 w-full mt-2 bg-white rounded-xl shadow-lg max-h-60 overflow-y-auto">
          <div class="p-2">
            <button 
              @click="filterByCategory(null)" 
              class="w-full text-left px-3 py-2 rounded hover:bg-red-50 text-gray-700"
              :class="selectedCategory === null ? 'font-semibold text-red-600 bg-red-50' : ''"
            >
              Все категории
            </button>
            <button 
              v-for="category in categories" 
              :key="category"
              @click="filterByCategory(category)"
              class="w-full text-left px-3 py-2 rounded hover:bg-red-50 text-gray-700"
              :class="selectedCategory === category ? 'font-semibold text-red-600 bg-red-50' : ''"
            >
              {{ category }}
            </button>
          </div>
        </div>
      </div>
      
      <!-- Переключатель избранного -->
      <button 
        @click="showOnlyFavorites = !showOnlyFavorites"
        class="px-4 py-3 bg-white rounded-xl shadow-md hover:shadow-lg transition-shadow flex items-center gap-2"
        :class="showOnlyFavorites ? 'bg-red-50 text-red-600 font-semibold' : 'text-gray-700'"
      >
        <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" :class="showOnlyFavorites ? 'text-red-600 fill-current' : 'text-gray-400'" viewBox="0 0 20 20" fill="currentColor">
          <path fill-rule="evenodd" d="M3.172 5.172a4 4 0 015.656 0L10 6.343l1.172-1.171a4 4 0 115.656 5.656L10 17.657l-6.828-6.829a4 4 0 010-5.656z" clip-rule="evenodd" />
        </svg>
        {{ showOnlyFavorites ? 'Все препараты' : 'Только избранные' }}
      </button>
    </div>

    <!-- Режим карточек -->
    <transition name="fade" mode="out-in">
      <div v-if="mode === 'cards'" class="mt-8">
        <!-- Поиск -->
        <div class="mb-6 max-w-md mx-auto">
          <div class="relative">
            <input 
              v-model="searchQuery" 
              type="text" 
              placeholder="Поиск препаратов..." 
              class="w-full px-4 py-3 rounded-xl border border-gray-300 focus:outline-none focus:ring-2 focus:ring-red-500 focus:border-transparent"
            />
            <button 
              v-if="searchQuery" 
              @click="searchQuery = ''" 
              class="absolute right-3 top-1/2 transform -translate-y-1/2 text-gray-400 hover:text-gray-600"
            >
              <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
                <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zM8.707 7.293a1 1 0 00-1.414 1.414L8.586 10l-1.293 1.293a1 1 0 101.414 1.414L10 11.414l1.293 1.293a1 1 0 001.414-1.414L11.414 10l1.293-1.293a1 1 0 00-1.414-1.414L10 8.586 8.707 7.293z" clip-rule="evenodd" />
              </svg>
            </button>
          </div>
        </div>

        <!-- Информация о количестве результатов -->
        <div class="text-center mb-6 text-gray-600">
          Найдено результатов: {{ filteredMedications.length }}
        </div>

        <!-- Сетка карточек -->
        <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
          <div v-for="med in filteredMedications" :key="med.id" 
            class="bg-white rounded-xl shadow-lg overflow-hidden hover:shadow-xl"
          >
            <div class="p-6">
              <div class="flex items-start justify-between">
                <h3 class="text-xl font-bold text-gray-800">{{ med.mnn }}</h3>
                <span class="bg-red-100 text-red-800 text-xs px-2 py-1 rounded-full">{{ med.category }}</span>
              </div>
              <p v-if="med.tradeName" class="text-gray-500 text-sm italic mt-1">{{ med.tradeName }}</p>
              
              <div class="mt-4 space-y-2">
                <div class="bg-gray-50 p-3 rounded-lg">
                  <h4 class="font-medium text-gray-700 mb-1">Форма выпуска:</h4>
                  <p class="text-sm text-gray-600">{{ med.form }}</p>
                </div>
                
                <div class="bg-gray-50 p-3 rounded-lg">
                  <h4 class="font-medium text-gray-700 mb-1">Дозировка:</h4>
                  <p class="text-sm text-gray-600">{{ med.dosage }}</p>
                </div>
              </div>
              
              <button 
                @click="toggleMedFavorite(med.id)"
                class="mt-4 inline-flex items-center text-sm font-medium text-red-600"
              >
                <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 mr-1" :class="isFavorite(med.id) ? 'text-red-600 fill-current' : 'text-gray-400'" viewBox="0 0 20 20" fill="currentColor">
                  <path fill-rule="evenodd" d="M3.172 5.172a4 4 0 015.656 0L10 6.343l1.172-1.171a4 4 0 115.656 5.656L10 17.657l-6.828-6.829a4 4 0 010-5.656z" clip-rule="evenodd" />
                </svg>
                {{ isFavorite(med.id) ? 'Избранное' : 'В избранное' }}
              </button>
            </div>
          </div>
        </div>

        <!-- Сообщение, если ничего не найдено -->
        <div v-if="filteredMedications.length === 0" class="text-center py-12">
          <div class="text-gray-400 mb-4">
            <svg xmlns="http://www.w3.org/2000/svg" class="h-16 w-16 mx-auto" fill="none" viewBox="0 0 24 24" stroke="currentColor">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9.172 16.172a4 4 0 015.656 0M9 10h.01M15 10h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
            </svg>
          </div>
          <p class="text-gray-600">По вашему запросу ничего не найдено.</p>
          <button @click="resetFilters" class="mt-4 px-4 py-2 bg-red-600 text-white rounded-full hover:bg-red-700 transition-colors">
            Сбросить фильтры
          </button>
        </div>
      </div>

      <!-- Режим тестирования -->
      <div v-else-if="mode === 'quiz'" class="max-w-2xl mx-auto">
        <div v-if="!quizCompleted" class="bg-white rounded-xl shadow-lg p-8">
          <div class="flex justify-between items-center mb-6">
            <h2 class="text-2xl font-bold text-gray-800">
              Вопрос {{ currentQuestionIndex + 1 }} / {{ quizQuestions.length }}
            </h2>
            <div class="text-sm text-gray-600">
              Правильных ответов: {{ correctAnswersCount }}
            </div>
          </div>

          <div class="quiz-question">
            <p class="text-xl font-medium mb-6">
              {{ currentQuestion.question }}
            </p>

            <div class="space-y-3">
              <div
                v-for="(option, index) in currentQuestion.options"
                :key="index"
                @click="selectAnswer(index)"
                class="w-full px-4 py-3 text-left rounded-lg transition-colors border-2 cursor-pointer"
                :class="getOptionClasses(index)"
              >
                {{ option }}
              </div>
            </div>

            <button 
              v-if="!answerSubmitted"
              @click="submitAnswer"
              :disabled="selectedAnswer === null"
              class="w-full mt-6 px-4 py-3 bg-gradient-to-r from-red-600 to-orange-500 text-white rounded-lg disabled:opacity-50 transition-colors"
            >
              Проверить ответ
            </button>

            <!-- Результат ответа -->
            <div 
              v-if="answerSubmitted"
              class="mt-6 p-4 rounded-lg transition-all duration-300"
              :class="isAnswerCorrect ? 'bg-green-50 border-2 border-green-300' : 'bg-red-50 border-2 border-red-300'"
            >
              <p class="font-semibold" 
                :class="isAnswerCorrect ? 'text-green-700' : 'text-red-700'"
              >
                {{ isAnswerCorrect ? 'Верно!' : 'Неверно!' }}
              </p>
              <p class="mt-2 text-gray-700">
                Правильный ответ: {{ currentQuestion.options[currentQuestion.correctAnswer] }}
              </p>
              <p class="mt-2 text-gray-700" v-if="currentQuestion.explanation">
                {{ currentQuestion.explanation }}
              </p>
            </div>
          </div>

          <!-- Навигация по вопросам -->
          <div class="flex justify-between mt-6">
            <button 
              @click="previousQuestion"
              :disabled="currentQuestionIndex === 0"
              class="px-4 py-2 rounded-lg bg-gray-100 text-gray-700 disabled:opacity-50"
            >
              Предыдущий
            </button>
            <button 
              @click="nextQuestion"
              :disabled="!answerSubmitted"
              class="px-4 py-2 rounded-lg bg-gradient-to-r from-red-600 to-orange-500 text-white disabled:opacity-50"
            >
              {{ currentQuestionIndex === quizQuestions.length - 1 ? 'Завершить' : 'Следующий' }}
            </button>
          </div>
        </div>

        <!-- Результаты теста -->
        <div v-else class="bg-white rounded-xl shadow-lg p-8 text-center">
          <h2 class="text-3xl font-bold text-gray-800 mb-4">
            Тест завершен!
          </h2>
          <div class="text-xl mb-6">
            Правильных ответов: 
            <span class="font-bold text-red-600">
              {{ correctAnswersCount }} из {{ quizQuestions.length }}
            </span>
          </div>
          <div class="text-2xl font-bold text-transparent bg-clip-text bg-gradient-to-r from-red-600 to-orange-500 mb-6">
            Результат: {{ Math.round((correctAnswersCount / quizQuestions.length) * 100) }}%
          </div>
          <button 
            @click="restartQuiz"
            class="px-6 py-3 bg-gradient-to-r from-red-600 to-orange-500 text-white rounded-full hover:scale-105 transition-transform"
          >
            Пройти тест заново
          </button>
        </div>
      </div>
    </transition>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onBeforeUnmount } from 'vue'
import { emergencyMeds } from '~/data/emergencyMeds.js'

// Состояние компонента
const mode = ref('cards')
const selectedCategory = ref(null)
const dropdownOpen = ref(false)
const searchQuery = ref('')
const favorites = ref([])
const showOnlyFavorites = ref(false)

// Обработка выпадающего списка категорий
const toggleCategoryDropdown = (event) => {
  event.stopPropagation()
  dropdownOpen.value = !dropdownOpen.value
}

// Закрытие выпадающего списка при клике вне его
const closeDropdownOnOutsideClick = (event) => {
  if (dropdownOpen.value) {
    dropdownOpen.value = false
  }
}

onMounted(() => {
  document.addEventListener('click', closeDropdownOnOutsideClick)
  
  // Загрузка избранных препаратов из localStorage
  const savedFavorites = localStorage.getItem('emergencyMedsFavorites')
  if (savedFavorites) {
    favorites.value = JSON.parse(savedFavorites)
  }
})

onBeforeUnmount(() => {
  document.removeEventListener('click', closeDropdownOnOutsideClick)
})

// Получение списка всех категорий
const categories = computed(() => {
  const categoriesSet = new Set(emergencyMeds.map(med => med.category))
  return [...categoriesSet].sort()
})

// Фильтрация препаратов по категории и поисковому запросу
const filteredMedications = computed(() => {
  let result = emergencyMeds

  // Фильтрация только избранных
  if (showOnlyFavorites.value) {
    result = result.filter(med => favorites.value.includes(med.id))
  }

  // Фильтрация по категории
  if (selectedCategory.value) {
    result = result.filter(med => med.category === selectedCategory.value)
  }

  // Фильтрация по поисковому запросу
  if (searchQuery.value) {
    const query = searchQuery.value.toLowerCase()
    result = result.filter(med => 
      med.mnn.toLowerCase().includes(query) ||
      (med.tradeName && med.tradeName.toLowerCase().includes(query)) ||
      med.form.toLowerCase().includes(query) ||
      med.category.toLowerCase().includes(query)
    )
  }

  return result
})

// Фильтрация по категории
const filterByCategory = (category) => {
  selectedCategory.value = category
  dropdownOpen.value = false
}

// Сброс всех фильтров
const resetFilters = () => {
  searchQuery.value = ''
  selectedCategory.value = null
}

// Управление избранными препаратами
const toggleMedFavorite = (id) => {
  const index = favorites.value.indexOf(id)
  if (index === -1) {
    favorites.value.push(id)
  } else {
    favorites.value.splice(index, 1)
  }
  
  // Сохранение в localStorage
  localStorage.setItem('emergencyMedsFavorites', JSON.stringify(favorites.value))
}

const isFavorite = (id) => {
  return favorites.value.includes(id)
}

// Логика для режима тестирования
const quizCompleted = ref(false)
const currentQuestionIndex = ref(0)
const selectedAnswer = ref(null)
const answerSubmitted = ref(false)
const userAnswers = ref([])
const correctAnswersCount = ref(0)

// Генерация случайных вопросов на основе данных о препаратах
const generateQuizQuestions = () => {
  // Для демонстрации генерируем простые вопросы на основе данных
  const questions = []
  
  // Вопросы о категории препарата
  for (let i = 0; i < 5; i++) {
    const randomMed = emergencyMeds[Math.floor(Math.random() * emergencyMeds.length)]
    const options = [randomMed.category]
    
    // Добавляем еще 3 случайные категории как варианты ответов
    while (options.length < 4) {
      const randomCategory = categories.value[Math.floor(Math.random() * categories.value.length)]
      if (!options.includes(randomCategory)) {
        options.push(randomCategory)
      }
    }
    
    // Перемешиваем варианты ответов
    const shuffledOptions = [...options].sort(() => Math.random() - 0.5)
    const correctAnswerIndex = shuffledOptions.indexOf(randomMed.category)
    
    questions.push({
      question: `К какой категории относится препарат ${randomMed.mnn}?`,
      options: shuffledOptions,
      correctAnswer: correctAnswerIndex
    })
  }
  
  // Вопросы о применении препаратов
  for (let i = 0; i < 5; i++) {
    const randomMed = emergencyMeds[Math.floor(Math.random() * emergencyMeds.length)]
    const correctMedication = randomMed.mnn
    
    // Собираем 3 других случайных препарата для вариантов
    const otherMeds = []
    while (otherMeds.length < 3) {
      const otherMed = emergencyMeds[Math.floor(Math.random() * emergencyMeds.length)]
      if (otherMed.mnn !== correctMedication && !otherMeds.includes(otherMed.mnn)) {
        otherMeds.push(otherMed.mnn)
      }
    }
    
    // Формируем вопрос на основе категории и формы выпуска
    const options = [correctMedication, ...otherMeds].sort(() => Math.random() - 0.5)
    const correctAnswerIndex = options.indexOf(correctMedication)
    
    questions.push({
      question: `Какой препарат относится к категории "${randomMed.category}" и выпускается в форме "${randomMed.form.split(';')[0]}"?`,
      options: options,
      correctAnswer: correctAnswerIndex,
      explanation: `${correctMedication} - ${randomMed.dosage}`
    })
  }
  
  return questions.sort(() => Math.random() - 0.5)
}

const quizQuestions = ref(generateQuizQuestions())

const currentQuestion = computed(() => {
  return quizQuestions.value[currentQuestionIndex.value]
})

const selectAnswer = (index) => {
  if (!answerSubmitted.value) {
    selectedAnswer.value = index
  }
}

const submitAnswer = () => {
  answerSubmitted.value = true
  userAnswers.value[currentQuestionIndex.value] = selectedAnswer.value
  
  if (selectedAnswer.value === currentQuestion.value.correctAnswer) {
    correctAnswersCount.value++
  }
}

const isAnswerCorrect = computed(() => {
  return selectedAnswer.value === currentQuestion.value.correctAnswer
})

const getOptionClasses = (index) => {
  if (!answerSubmitted.value) {
    return selectedAnswer.value === index
      ? 'border-red-400 bg-red-50'
      : 'border-gray-200 hover:border-red-300 hover:bg-red-50'
  }
  
  const isCorrect = index === currentQuestion.value.correctAnswer
  const isSelected = index === selectedAnswer.value
  
  if (isCorrect) {
    return 'border-green-500 bg-green-50'
  }
  
  if (isSelected && !isCorrect) {
    return 'border-red-500 bg-red-50'
  }
  
  return 'border-gray-200 opacity-60'
}

const nextQuestion = () => {
  if (currentQuestionIndex.value < quizQuestions.value.length - 1) {
    currentQuestionIndex.value++
    selectedAnswer.value = userAnswers.value[currentQuestionIndex.value] || null
    answerSubmitted.value = userAnswers.value[currentQuestionIndex.value] !== undefined
  } else {
    quizCompleted.value = true
  }
}

const previousQuestion = () => {
  if (currentQuestionIndex.value > 0) {
    currentQuestionIndex.value--
    selectedAnswer.value = userAnswers.value[currentQuestionIndex.value] || null
    answerSubmitted.value = userAnswers.value[currentQuestionIndex.value] !== undefined
  }
}

const restartQuiz = () => {
  quizQuestions.value = generateQuizQuestions()
  currentQuestionIndex.value = 0
  selectedAnswer.value = null
  answerSubmitted.value = false
  userAnswers.value = []
  correctAnswersCount.value = 0
  quizCompleted.value = false
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