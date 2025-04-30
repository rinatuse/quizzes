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
      <button 
        @click="currentMode = 'cards'"
        class="px-6 py-2 rounded-full transition-all duration-300 transform"
        :class="currentMode === 'cards' 
          ? 'bg-gradient-to-r from-purple-600 to-indigo-600 text-white scale-105 shadow-lg' 
          : 'bg-gray-100 text-gray-700 hover:bg-gray-200'"
      >
        Справочник
      </button>
      <button 
        @click="currentMode = 'pharmacology'"
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
        <div class="grid md:grid-cols-2 gap-6">
          <!-- Список категорий -->
          <div class="bg-white rounded-xl shadow-lg p-6">
            <h2 class="text-2xl font-bold text-indigo-700 mb-4">Категории</h2>
            <div class="space-y-2">
              <button 
                v-for="category in uniqueCategories" 
                :key="category"
                @click="selectCategory(category)"
                class="w-full text-left px-4 py-2 rounded-lg transition-all duration-300"
                :class="selectedCategory === category 
                  ? 'bg-indigo-100 text-indigo-700' 
                  : 'hover:bg-gray-100 text-gray-700'"
              >
                {{ category }}
              </button>
            </div>
          </div>

          <!-- Карточки препаратов -->
          <div class="space-y-6">
            <div 
              v-for="(med, index) in filteredMedications" 
              :key="index"
              class="bg-white rounded-xl shadow-lg p-6 hover:shadow-xl transition-all duration-300 transform hover:-translate-y-2 cursor-pointer"
              @click="openMedicationDetails(med)"
            >
              <h3 class="text-xl font-bold text-indigo-700 mb-2">{{ med.title }}</h3>
              <p class="text-gray-600 mb-4">{{ med.category }}</p>
              <div class="text-sm text-gray-500 font-mono">{{ med.text }}</div>
            </div>
          </div>
        </div>
      </div>

      <!-- Контейнер для обычного квиза (один ответ) -->
      <div v-else-if="currentMode === 'quiz'" class="quiz-container max-w-2xl mx-auto">
        <div v-if="!quizCompleted" class="bg-white rounded-xl shadow-lg p-8">
          <div class="flex justify-between items-center mb-6">
            <h2 class="text-2xl font-bold text-indigo-700">
              Вопрос {{ currentQuestionIndex + 1 }} / {{ quizzes.length }}
            </h2>
            <div class="text-sm text-gray-600">
              Правильных ответов: {{ correctAnswersCount }}
            </div>
          </div>

          <div class="quiz-question">
            <p class="text-xl font-medium mb-6">
              {{ currentQuestion.question }}
            </p>

            <div class="space-y-4">
              <button
                v-for="(option, index) in currentQuestion.options"
                :key="index"
                @click="selectAnswer(index)"
                class="w-full px-4 py-3 text-left rounded-lg transition-all duration-300 border-2"
                :class="getOptionClasses(index)"
              >
                {{ option }}
              </button>
            </div>

            <!-- Результат ответа -->
            <div 
              v-if="answerSelected"
              class="mt-6 p-4 rounded-lg transition-all duration-300"
              :class="isCurrentAnswerCorrect 
                ? 'bg-green-50 border-2 border-green-300' 
                : 'bg-red-50 border-2 border-red-300'"
            >
              <p class="font-semibold" 
                :class="isCurrentAnswerCorrect 
                  ? 'text-green-700' 
                  : 'text-red-700'"
              >
                {{ isCurrentAnswerCorrect ? 'Верно!' : 'Неверно!' }}
              </p>
              <p class="mt-2 text-gray-700">
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
              :disabled="!answerSelected"
              class="px-4 py-2 rounded-lg bg-indigo-600 text-white disabled:opacity-50"
            >
              {{ currentQuestionIndex === quizzes.length - 1 ? 'Завершить' : 'Следующий' }}
            </button>
          </div>
        </div>

        <!-- Результаты квиза -->
        <div v-else class="bg-white rounded-xl shadow-lg p-8 text-center">
          <h2 class="text-3xl font-bold text-indigo-700 mb-4">
            Тест завершен!
          </h2>
          <div class="text-xl mb-6">
            Правильных ответов: 
            <span class="font-bold text-indigo-600">
              {{ correctAnswersCount }} из {{ quizzes.length }}
            </span>
          </div>
          <div class="text-2xl font-bold text-transparent bg-clip-text bg-gradient-to-r from-purple-600 to-indigo-600 mb-6">
            Результат: {{ Math.round((correctAnswersCount / quizzes.length) * 100) }}%
          </div>
          <button 
            @click="restartQuiz"
            class="px-6 py-3 bg-gradient-to-r from-purple-600 to-indigo-600 text-white rounded-full hover:scale-105 transition-transform"
          >
            Пройти тест заново
          </button>
        </div>
      </div>

      <!-- Контейнер для фармакологического квиза (множественный выбор) -->
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
import { ref, computed } from 'vue'
import { prescriptions } from '~/data/prescriptions.js'
import { quizzes } from '~/data/quizzes.js'

// Состояние приложения
const currentMode = ref('cards')
const selectedCategory = ref(null)
const selectedMedication = ref(null)

// Квиз
const currentQuestionIndex = ref(0)
const selectedAnswers = ref([])
const quizCompleted = ref(false)
const answerSelected = ref(false)

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
  selectedCategory.value = selectedCategory.value === category ? null : category
}

const openMedicationDetails = (medication) => {
  selectedMedication.value = medication
}

// Методы для квиза
const currentQuestion = computed(() => quizzes[currentQuestionIndex.value])

const isCurrentAnswerCorrect = computed(() => {
  return selectedAnswers.value[currentQuestionIndex.value] === currentQuestion.value.correctAnswer
})

const correctAnswersCount = computed(() => {
  return selectedAnswers.value.filter((answer, index) => 
    answer === quizzes[index].correctAnswer
  ).length
})

const selectAnswer = (index) => {
  if (!answerSelected.value) {
    selectedAnswers.value[currentQuestionIndex.value] = index
    answerSelected.value = true
  }
}

const getOptionClasses = (index) => {
  if (!answerSelected.value) {
    return 'border-gray-200 hover:bg-gray-50'
  }
  
  if (index === currentQuestion.value.correctAnswer) {
    return 'border-green-300 bg-green-50 text-green-700'
  }
  
  if (index === selectedAnswers.value[currentQuestionIndex.value]) {
    return 'border-red-300 bg-red-50 text-red-700'
  }
  
  return 'border-gray-200 opacity-50'
}

const nextQuestion = () => {
  if (currentQuestionIndex.value < quizzes.length - 1) {
    currentQuestionIndex.value++
    answerSelected.value = selectedAnswers.value[currentQuestionIndex.value] !== undefined
  } else {
    quizCompleted.value = true
  }
}

const previousQuestion = () => {
  if (currentQuestionIndex.value > 0) {
    currentQuestionIndex.value--
    answerSelected.value = selectedAnswers.value[currentQuestionIndex.value] !== undefined
  }
}

const restartQuiz = () => {
  currentQuestionIndex.value = 0
  selectedAnswers.value = []
  quizCompleted.value = false
  answerSelected.value = false
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