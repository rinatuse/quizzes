<template>
  <div class="container mx-auto px-4 py-8">
    <!-- Заголовок и описание -->
    <div class="text-center mb-10">
      <h1 class="text-4xl font-extrabold text-transparent bg-clip-text bg-gradient-to-r from-purple-600 to-indigo-600 mb-4">
        Медицинский Справочник
      </h1>
      <p class="text-gray-600 max-w-2xl mx-auto">
        Интерактивное приложение для изучения фармацевтических препаратов и проверки знаний
      </p>
    </div>

    <!-- Навигация между режимами -->
    <div class="flex justify-center space-x-4 mb-8">
      <div class="relative">
        <button 
          @click="toggleDropdown"
          class="px-6 py-2 rounded-full transition-all duration-300 transform flex items-center space-x-2"
          :class="currentMode === 'cards' 
            ? 'bg-gradient-to-r from-purple-600 to-indigo-600 text-white scale-105 shadow-lg' 
            : 'bg-gray-100 text-gray-700 hover:bg-gray-200'"
        >
          <span>Справочник</span>
          <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" :class="dropdownOpen ? 'transform rotate-180' : ''" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 9l-7 7-7-7" />
          </svg>
        </button>
        
        <!-- Выпадающий список категорий -->
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
            v-for="category in uniqueCategories" 
            :key="category"
            @click="selectCategory(category)"
            class="w-full text-left px-4 py-2 hover:bg-indigo-50 text-gray-700"
            :class="selectedCategory === category ? 'font-bold text-indigo-700 bg-indigo-50' : ''"
          >
            {{ category }}
          </button>
        </div>
      </div>
      
      <button 
        @click="switchToPharmacology"
        class="px-6 py-2 rounded-full transition-all duration-300 transform"
        :class="currentMode === 'pharmacology' 
          ? 'bg-gradient-to-r from-purple-600 to-indigo-600 text-white scale-105 shadow-lg' 
          : 'bg-gray-100 text-gray-700 hover:bg-gray-200'"
      >
        Фармакология
      </button>
    </div>

    <!-- Контейнер для карточек -->
    <transition name="fade" mode="out-in">
      <div v-if="currentMode === 'cards'" class="cards-container">
        <!-- Информация о текущей категории -->
        <div class="mb-6 text-center" v-if="selectedCategory">
          <h2 class="text-2xl font-bold text-indigo-700">{{ selectedCategory }}</h2>
          <button 
            @click="selectedCategory = null"
            class="text-sm text-indigo-600 hover:text-indigo-800 mt-2"
          >
            Показать все категории
          </button>
        </div>
        <div v-else class="mb-6 text-center">
          <h2 class="text-2xl font-bold text-indigo-700">Все препараты</h2>
        </div>

        <!-- Сетка карточек -->
        <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
          <div 
            v-for="(med, index) in filteredMedications" 
            :key="index"
            class="bg-white rounded-xl shadow-lg p-6 hover:shadow-xl transition-all duration-300 transform hover:-translate-y-2 cursor-pointer h-full flex flex-col"
            @click="openMedicationDetails(med)"
          >
            <h3 class="text-xl font-bold text-indigo-700 mb-2">{{ med.title }}</h3>
            <p class="text-gray-600 mb-4">{{ med.category }}</p>
            <div class="text-sm text-gray-500 font-mono mt-auto">{{ med.text }}</div>
          </div>
        </div>

        <!-- Сообщение если нет результатов -->
        <div v-if="filteredMedications.length === 0" class="text-center py-10">
          <p class="text-gray-600">Нет препаратов в выбранной категории.</p>
        </div>
      </div>

      <!-- Контейнер для фармакологического квиза (многовариантный выбор) -->
      <div v-else-if="currentMode === 'pharmacology'" class="quiz-container max-w-2xl mx-auto">
        <div v-if="!pharmacologyQuizCompleted" class="bg-white rounded-xl shadow-lg p-8">
          <div class="flex justify-between items-center mb-6">
            <h2 class="text-2xl font-bold text-indigo-700">
              Вопрос {{ currentPharmQuestionIndex + 1 }} / {{ pharmacologyQuizzes.length }}
            </h2>
            <div class="text-sm text-gray-600">
              Правильных ответов: {{ pharmacologyCorrectAnswersCount }}
            </div>
          </div>

          <div class="quiz-question">
            <p class="text-xl font-medium mb-6">
              {{ currentPharmQuestion.question }}
            </p>

            <div class="space-y-4">
              <div
                v-for="(option, index) in currentPharmQuestion.options"
                :key="index"
                @click="togglePharmAnswer(index)"
                class="w-full px-4 py-3 text-left rounded-lg transition-all duration-300 border-2 flex items-center cursor-pointer"
                :class="getPharmOptionClasses(index)"
              >
                <div class="w-6 h-6 border-2 rounded mr-3 flex items-center justify-center"
                  :class="getPharmCheckboxClasses(index)"
                >
                  <svg v-if="isPharmOptionSelected(index)" xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7" />
                  </svg>
                </div>
                {{ option }}
              </div>
            </div>

            <button 
              v-if="!pharmAnswerSubmitted"
              @click="submitPharmAnswer"
              :disabled="currentPharmSelectedAnswers.length === 0"
              class="w-full mt-6 px-4 py-3 bg-indigo-600 text-white rounded-lg disabled:opacity-50 transition-all duration-300"
            >
              Проверить ответ
            </button>

            <!-- Результат ответа -->
            <div 
              v-if="pharmAnswerSubmitted"
              class="mt-6 p-4 rounded-lg transition-all duration-300"
              :class="isCurrentPharmAnswerCorrect 
                ? 'bg-green-50 border-2 border-green-300' 
                : 'bg-red-50 border-2 border-red-300'"
            >
              <p class="font-semibold" 
                :class="isCurrentPharmAnswerCorrect 
                  ? 'text-green-700' 
                  : 'text-red-700'"
              >
                {{ isCurrentPharmAnswerCorrect ? 'Верно!' : 'Неверно!' }}
              </p>
              <p class="mt-2 text-gray-700">
                {{ currentPharmQuestion.explanation }}
              </p>
            </div>
          </div>

          <!-- Навигация по вопросам -->
          <div class="flex justify-between mt-6">
            <button 
              @click="previousPharmQuestion"
              :disabled="currentPharmQuestionIndex === 0"
              class="px-4 py-2 rounded-lg bg-gray-100 text-gray-700 disabled:opacity-50"
            >
              Предыдущий
            </button>
            <button 
              @click="nextPharmQuestion"
              :disabled="!pharmAnswerSubmitted"
              class="px-4 py-2 rounded-lg bg-indigo-600 text-white disabled:opacity-50"
            >
              {{ currentPharmQuestionIndex === pharmacologyQuizzes.length - 1 ? 'Завершить' : 'Следующий' }}
            </button>
          </div>
        </div>

        <!-- Результаты квиза по фармакологии -->
        <div v-else class="bg-white rounded-xl shadow-lg p-8 text-center">
          <h2 class="text-3xl font-bold text-indigo-700 mb-4">
            Тест по фармакологии завершен!
          </h2>
          <div class="text-xl mb-6">
            Правильных ответов: 
            <span class="font-bold text-indigo-600">
              {{ pharmacologyCorrectAnswersCount }} из {{ pharmacologyQuizzes.length }}
            </span>
          </div>
          <div class="text-2xl font-bold text-transparent bg-clip-text bg-gradient-to-r from-purple-600 to-indigo-600 mb-6">
            Результат: {{ Math.round((pharmacologyCorrectAnswersCount / pharmacologyQuizzes.length) * 100) }}%
          </div>
          <button 
            @click="restartPharmQuiz"
            class="px-6 py-3 bg-gradient-to-r from-purple-600 to-indigo-600 text-white rounded-full hover:scale-105 transition-transform"
          >
            Пройти тест заново
          </button>
        </div>
      </div>
    </transition>

    <!-- Модальное окно для деталей препарата -->
    <div 
      v-if="selectedMedication" 
      class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50 p-4"
      @click.self="selectedMedication = null"
    >
      <div 
        class="bg-white rounded-xl shadow-2xl max-w-2xl w-full max-h-[90vh] overflow-y-auto p-8 relative"
        @click.stop
      >
        <button 
          @click="selectedMedication = null"
          class="absolute top-4 right-4 text-gray-500 hover:text-gray-700"
        >
          ✕
        </button>
        <h2 class="text-3xl font-bold text-indigo-700 mb-4">
          {{ selectedMedication.title }}
        </h2>
        <div class="space-y-4">
          <div>
            <h3 class="font-semibold text-gray-700">Рецепт:</h3>
            <pre class="bg-gray-100 p-4 rounded-lg text-sm">{{ selectedMedication.text }}</pre>
          </div>
          <div>
            <h3 class="font-semibold text-gray-700">Описание:</h3>
            <p class="text-gray-600">{{ selectedMedication.description }}</p>
          </div>
          <div>
            <h3 class="font-semibold text-gray-700">Ключевые моменты:</h3>
            <ul class="list-disc list-inside text-gray-600">
              <li v-for="(point, index) in selectedMedication.keyPoints" :key="index">
                {{ point }}
              </li>
            </ul>
          </div>
          <div>
            <h3 class="font-semibold text-gray-700">Категория:</h3>
            <span class="bg-indigo-100 text-indigo-700 px-3 py-1 rounded-full">
              {{ selectedMedication.category }}
            </span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onBeforeUnmount } from 'vue'
import { prescriptions } from '~/data/prescriptions.js'
import { quizzes } from '~/data/quizzes.js'

// Состояние приложения
const currentMode = ref('cards')
const selectedCategory = ref(null)
const selectedMedication = ref(null)
const dropdownOpen = ref(false)

// Обработчик кликов вне выпадающего списка
const closeDropdownOnOutsideClick = (event) => {
  if (dropdownOpen.value) {
    dropdownOpen.value = false
  }
}

// Добавляем слушатель события при монтировании компонента
onMounted(() => {
  document.addEventListener('click', closeDropdownOnOutsideClick)
})

// Удаляем слушатель события перед размонтированием компонента
onBeforeUnmount(() => {
  document.removeEventListener('click', closeDropdownOnOutsideClick)
})

// Фармакологический квиз (многовариантный выбор)
const currentPharmQuestionIndex = ref(0)
const pharmSelectedAnswers = ref([]) // Массив массивов выбранных ответов для каждого вопроса
const pharmacologyQuizCompleted = ref(false)
const pharmAnswerSubmitted = ref(false)

// Модифицируем структуру данных quizzes для поддержки множественного выбора
const pharmacologyQuizzes = computed(() => {
  return quizzes.map(question => {
    // Преобразуем формат вопроса для поддержки множественного выбора
    // Если у вопроса уже есть поле correctAnswers, используем его
    // Иначе создаем массив с одним правильным ответом из correctAnswer
    return {
      ...question,
      correctAnswers: question.correctAnswers || [question.correctAnswer]
    }
  })
})

// Инициализация массива ответов для фармакологического квиза
const initPharmSelectedAnswers = () => {
  pharmSelectedAnswers.value = Array(pharmacologyQuizzes.value.length).fill().map(() => [])
}

// Инициализируем массив при загрузке компонента
initPharmSelectedAnswers()

// Методы для работы с выпадающим списком
const toggleDropdown = (event) => {
  event.stopPropagation() // Предотвращаем всплытие события
  dropdownOpen.value = !dropdownOpen.value
  if (currentMode.value !== 'cards') {
    currentMode.value = 'cards'
  }
}

// Методы для навигации по режимам
const switchToPharmacology = () => {
  currentMode.value = 'pharmacology'
  dropdownOpen.value = false
}

// Computed свойства для карточек
const uniqueCategories = computed(() => {
  return [...new Set(prescriptions.map(p => p.category))]
})

const filteredMedications = computed(() => {
  return selectedCategory.value
    ? prescriptions.filter(med => med.category === selectedCategory.value)
    : prescriptions
})

// Методы для работы с карточками
const selectCategory = (category) => {
  selectedCategory.value = category
  dropdownOpen.value = false
  currentMode.value = 'cards'
}

const openMedicationDetails = (medication) => {
  selectedMedication.value = medication
}

// Методы для фармакологического квиза (многовариантный выбор)
const currentPharmQuestion = computed(() => pharmacologyQuizzes.value[currentPharmQuestionIndex.value])

const currentPharmSelectedAnswers = computed(() => 
  pharmSelectedAnswers.value[currentPharmQuestionIndex.value] || []
)

const isPharmOptionSelected = (optionIndex) => {
  return currentPharmSelectedAnswers.value.includes(optionIndex)
}

const togglePharmAnswer = (optionIndex) => {
  if (pharmAnswerSubmitted.value) return // Нельзя изменять ответ после отправки

  const currentAnswers = [...currentPharmSelectedAnswers.value]
  const answerIndex = currentAnswers.indexOf(optionIndex)
  
  if (answerIndex === -1) {
    currentAnswers.push(optionIndex)
  } else {
    currentAnswers.splice(answerIndex, 1)
  }

  // Обновляем выбранные ответы
  pharmSelectedAnswers.value[currentPharmQuestionIndex.value] = currentAnswers
}

const submitPharmAnswer = () => {
  pharmAnswerSubmitted.value = true
}

const isCurrentPharmAnswerCorrect = computed(() => {
  const correctAnswers = currentPharmQuestion.value.correctAnswers || [currentPharmQuestion.value.correctAnswer];
  const selectedAnswers = currentPharmSelectedAnswers.value;
  
  // Если количество выбранных ответов не совпадает с количеством правильных, то ответ неверный
  if (correctAnswers.length !== selectedAnswers.length) {
    return false;
  }
  
  // Сортируем массивы для корректного сравнения
  const sortedCorrect = [...correctAnswers].sort();
  const sortedSelected = [...selectedAnswers].sort();
  
  // Проверяем, что все элементы совпадают
  for (let i = 0; i < sortedCorrect.length; i++) {
    if (sortedCorrect[i] !== sortedSelected[i]) {
      return false;
    }
  }
  
  return true;
})

const pharmacologyCorrectAnswersCount = computed(() => {
  // Считаем количество правильных ответов
  let count = 0
  for (let i = 0; i < pharmacologyQuizzes.value.length; i++) {
    const correctAnswers = pharmacologyQuizzes.value[i].correctAnswers || [pharmacologyQuizzes.value[i].correctAnswer];
    const selectedAnswers = pharmSelectedAnswers.value[i] || []
    
    // Если количество выбранных ответов не совпадает с количеством правильных, то ответ неверный
    if (correctAnswers.length !== selectedAnswers.length) {
      continue;
    }
    
    // Сортируем массивы для корректного сравнения
    const sortedCorrect = [...correctAnswers].sort();
    const sortedSelected = [...selectedAnswers].sort();
    
    // Проверяем, что все элементы совпадают
    let allCorrect = true;
    for (let j = 0; j < sortedCorrect.length; j++) {
      if (sortedCorrect[j] !== sortedSelected[j]) {
        allCorrect = false;
        break;
      }
    }
    
    if (allCorrect) {
      count++;
    }
  }
  return count
})

const getPharmOptionClasses = (optionIndex) => {
  if (!pharmAnswerSubmitted.value) {
    return isPharmOptionSelected(optionIndex)
      ? 'border-indigo-300 bg-indigo-50'
      : 'border-gray-200 hover:bg-gray-50'
  }
  
  const isCorrectOption = currentPharmQuestion.value.correctAnswers.includes(optionIndex)
  const isSelected = isPharmOptionSelected(optionIndex)
  
  if (isCorrectOption) {
    return 'border-green-300 bg-green-50'
  }
  
  if (isSelected && !isCorrectOption) {
    return 'border-red-300 bg-red-50'
  }
  
  return 'border-gray-200 opacity-50'
}

const getPharmCheckboxClasses = (optionIndex) => {
  if (!pharmAnswerSubmitted.value) {
    return isPharmOptionSelected(optionIndex)
      ? 'border-indigo-500 bg-indigo-100 text-indigo-700'
      : 'border-gray-300'
  }
  
  const isCorrectOption = currentPharmQuestion.value.correctAnswers.includes(optionIndex)
  const isSelected = isPharmOptionSelected(optionIndex)
  
  if (isCorrectOption) {
    return 'border-green-500 bg-green-100 text-green-700'
  }
  
  if (isSelected && !isCorrectOption) {
    return 'border-red-500 bg-red-100 text-red-700'
  }
  
  return 'border-gray-300'
}

const nextPharmQuestion = () => {
  if (currentPharmQuestionIndex.value < pharmacologyQuizzes.value.length - 1) {
    currentPharmQuestionIndex.value++
    pharmAnswerSubmitted.value = pharmSelectedAnswers.value[currentPharmQuestionIndex.value] && 
                                 pharmSelectedAnswers.value[currentPharmQuestionIndex.value].length > 0
  } else {
    pharmacologyQuizCompleted.value = true
  }
}

const previousPharmQuestion = () => {
  if (currentPharmQuestionIndex.value > 0) {
    currentPharmQuestionIndex.value--
    pharmAnswerSubmitted.value = pharmSelectedAnswers.value[currentPharmQuestionIndex.value] && 
                                pharmSelectedAnswers.value[currentPharmQuestionIndex.value].length > 0
  }
}

const restartPharmQuiz = () => {
  currentPharmQuestionIndex.value = 0
  initPharmSelectedAnswers()
  pharmacologyQuizCompleted.value = false
  pharmAnswerSubmitted.value = false
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