<template>
  <div class="my-8">
    <!-- Режимы работы -->
    <div class="flex justify-center gap-2 mb-6">
      <button 
        @click="mode = 'cards'" 
        class="px-4 py-1 transition-all duration-300 border"
        :class="mode === 'cards' ? 'border-gray-500 bg-white' : 'border-gray-300 bg-gray-100 hover:bg-white'"
      >
        Карточки
      </button>
      <button 
        @click="mode = 'test'" 
        class="px-4 py-1 transition-all duration-300 border"
        :class="mode === 'test' ? 'border-gray-500 bg-white' : 'border-gray-300 bg-gray-100 hover:bg-white'"
      >
        Тесты
      </button>
    </div>

    <!-- Режим карточек -->
    <div v-if="mode === 'cards'" class="cards-container">
      <div class="text-left mb-4">
        <h2 class="text-lg font-medium mb-3">{{ currentIndex + 1 }} из {{ prescriptions.length }}</h2>
        <div class="flex gap-2 mb-4">
          <button 
            @click="prevCard" 
            class="px-2 py-1 border border-gray-300 transition-all hover:bg-gray-50"
            :disabled="currentIndex === 0"
            :class="{ 'opacity-50 cursor-not-allowed': currentIndex === 0 }"
          >
            ← Пред
          </button>
          <button 
            @click="nextCard" 
            class="px-2 py-1 border border-gray-300 transition-all hover:bg-gray-50"
            :disabled="currentIndex === prescriptions.length - 1"
            :class="{ 'opacity-50 cursor-not-allowed': currentIndex === prescriptions.length - 1 }"
          >
            След →
          </button>
        </div>
      </div>

      <transition name="card-fade" mode="out-in">
        <div 
          :key="'front-' + currentIndex" 
          v-if="!showAnswer" 
          class="card bg-white shadow border border-gray-200 p-5 max-w-2xl mx-auto cursor-pointer"
          @click="toggleCard"
        >
          <h3 class="text-xl font-bold text-gray-800 mb-4">{{ currentCard.title }}</h3>
          <div class="whitespace-pre-line text-lg font-mono">{{ currentCard.text }}</div>
          <p class="text-center mt-4 text-gray-500">(Нажмите для просмотра информации)</p>
        </div>
        
        <div 
          :key="'back-' + currentIndex" 
          v-else 
          class="card bg-white shadow border border-gray-200 p-5 max-w-2xl mx-auto cursor-pointer"
          @click="toggleCard"
        >
          <h3 class="text-xl font-bold text-gray-800 mb-4">{{ currentCard.title }}</h3>
          <p class="text-gray-700 mb-4">{{ currentCard.description }}</p>
          
          <h4 class="font-semibold text-gray-700 mt-4 mb-2">Ключевые моменты:</h4>
          <ul class="list-disc pl-5">
            <li v-for="(point, idx) in currentCard.keyPoints" :key="idx" class="mb-1">
              {{ point }}
            </li>
          </ul>
          
          <p class="font-semibold mt-4">
            Категория: {{ currentCard.category }}
          </p>
          
          <p class="text-center mt-4 text-gray-500">(Нажмите для просмотра рецепта)</p>
        </div>
      </transition>
    </div>

    <!-- Режим тестов -->
    <div v-else-if="mode === 'test'" class="test-container max-w-2xl mx-auto">
      <transition name="fade" mode="out-in">
        <div v-if="!testCompleted" :key="'question-' + currentQuestionIndex">
          <h2 class="text-lg font-medium mb-3">Вопрос {{ currentQuestionIndex + 1 }} из {{ quizzes.length }}</h2>
          <div class="bg-white shadow border border-gray-200 p-5 mb-5">
            <h3 class="text-lg font-medium mb-4">{{ currentQuestion.question }}</h3>
            
            <div class="space-y-2">
              <div 
                v-for="(option, idx) in currentQuestion.options" 
                :key="idx"
                class="option border p-2 cursor-pointer transition-all"
                :class="getOptionClass(idx)"
                @click="selectAnswer(idx)"
              >
                {{ option }}
              </div>
            </div>
            
            <div v-if="answerSelected" class="mt-5 p-3 border" :class="isCorrect ? 'border-green-300 bg-green-50' : 'border-red-300 bg-red-50'">
              <p class="font-medium" :class="isCorrect ? 'text-green-800' : 'text-red-800'">
                {{ isCorrect ? 'Правильно!' : 'Неправильно!' }}
              </p>
              <p class="mt-2">{{ currentQuestion.explanation }}</p>
            </div>
          </div>
          
          <div class="flex justify-between">
            <button 
              @click="prevQuestion" 
              class="px-2 py-1 border border-gray-300 transition-all hover:bg-gray-50"
              :disabled="currentQuestionIndex === 0"
              :class="{ 'opacity-50 cursor-not-allowed': currentQuestionIndex === 0 }"
            >
              ← Предыдущий
            </button>
            
            <button 
              v-if="answerSelected"
              @click="nextQuestion" 
              class="px-2 py-1 border border-gray-300 transition-all hover:bg-gray-50"
              :disabled="currentQuestionIndex === quizzes.length - 1 && answerSelected"
            >
              {{ currentQuestionIndex === quizzes.length - 1 ? 'Завершить тест' : 'Следующий →' }}
            </button>
          </div>
        </div>
        
        <div v-else :key="'results'">
          <div class="bg-white shadow border border-gray-200 p-5 text-center">
            <h2 class="text-xl font-bold mb-4">Тест завершен!</h2>
            <p class="text-lg mb-3">Вы ответили правильно на {{ correctAnswers }} из {{ quizzes.length }} вопросов.</p>
            <p class="text-lg">Ваш результат: {{ Math.round((correctAnswers / quizzes.length) * 100) }}%</p>
            
            <button 
              @click="resetTest" 
              class="mt-5 px-4 py-1 border border-gray-300 transition-all hover:bg-gray-50"
            >
              Пройти тест снова
            </button>
          </div>
        </div>
      </transition>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue';
import gsap from 'gsap';
import { prescriptions } from '~/data/prescriptions.js';
import { quizzes } from '~/data/quizzes.js';


// Состояние приложения
const mode = ref('cards');
const currentIndex = ref(0);
const showAnswer = ref(false);
const currentQuestionIndex = ref(0);
const selectedAnswers = ref(new Array(quizzes.length).fill(null));
const answerSelected = ref(false);
const testCompleted = ref(false);
const correctAnswers = ref(0);

// Вычисляемые свойства
const currentCard = computed(() => prescriptions[currentIndex.value]);
const currentQuestion = computed(() => quizzes[currentQuestionIndex.value]);
const isCorrect = computed(() => {
  if (selectedAnswers.value[currentQuestionIndex.value] === null) return false;
  return selectedAnswers.value[currentQuestionIndex.value] === currentQuestion.value.correctAnswer;
});

// Методы для работы с карточками
const nextCard = () => {
  if (currentIndex.value < prescriptions.length - 1) {
    showAnswer.value = false;
    gsap.to('.card', {
      y: -10,
      opacity: 0,
      duration: 0.2,
      onComplete: () => {
        currentIndex.value++;
        gsap.to('.card', {
          y: 0,
          opacity: 1,
          duration: 0.3
        });
      }
    });
  }
};

const prevCard = () => {
  if (currentIndex.value > 0) {
    showAnswer.value = false;
    gsap.to('.card', {
      y: 10,
      opacity: 0,
      duration: 0.2,
      onComplete: () => {
        currentIndex.value--;
        gsap.to('.card', {
          y: 0,
          opacity: 1,
          duration: 0.3
        });
      }
    });
  }
};

const toggleCard = () => {
  showAnswer.value = !showAnswer.value;
};

// Методы для работы с тестами
const selectAnswer = (index) => {
  if (!answerSelected.value) {
    selectedAnswers.value[currentQuestionIndex.value] = index;
    answerSelected.value = true;
    
    if (index === currentQuestion.value.correctAnswer) {
      correctAnswers.value++;
    }
  }
};

const getOptionClass = (index) => {
  if (!answerSelected.value) {
    return 'hover:bg-gray-50';
  }
  
  if (index === currentQuestion.value.correctAnswer) {
    return 'bg-green-50 border-green-300';
  }
  
  if (index === selectedAnswers.value[currentQuestionIndex.value] && 
      index !== currentQuestion.value.correctAnswer) {
    return 'bg-red-50 border-red-300';
  }
  
  return 'opacity-70';
};

const nextQuestion = () => {
  if (currentQuestionIndex.value < quizzes.length - 1) {
    gsap.to('.test-container > div', {
      x: -20,
      opacity: 0,
      duration: 0.2,
      onComplete: () => {
        currentQuestionIndex.value++;
        answerSelected.value = selectedAnswers.value[currentQuestionIndex.value] !== null;
        gsap.to('.test-container > div', {
          x: 0,
          opacity: 1,
          duration: 0.3
        });
      }
    });
  } else {
    testCompleted.value = true;
  }
};

const prevQuestion = () => {
  if (currentQuestionIndex.value > 0) {
    gsap.to('.test-container > div', {
      x: 20,
      opacity: 0,
      duration: 0.2,
      onComplete: () => {
        currentQuestionIndex.value--;
        answerSelected.value = selectedAnswers.value[currentQuestionIndex.value] !== null;
        gsap.to('.test-container > div', {
          x: 0,
          opacity: 1,
          duration: 0.3
        });
      }
    });
  }
};

const resetTest = () => {
  selectedAnswers.value = new Array(quizzes.length).fill(null);
  currentQuestionIndex.value = 0;
  answerSelected.value = false;
  testCompleted.value = false;
  correctAnswers.value = 0;
};

// Анимация компонентов
onMounted(() => {
  gsap.from('.cards-container, .test-container', {
    y: 30,
    opacity: 0,
    duration: 0.5
  });
});
</script>

<style scoped>
/* Анимации для карточек */
.card-fade-enter-active,
.card-fade-leave-active {
  transition: opacity 0.4s ease;
}

.card-fade-enter-from,
.card-fade-leave-to {
  opacity: 0;
}

/* Анимации для смены вопросов */
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

/* Стили для карточек */
.card {
  transition: all 0.2s ease;
}

.card:hover {
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

/* Стили для опций в тесте */
.option {
  border-color: #e0e0e0;
}

.option:hover {
  border-color: #999;
  background-color: #f9f9f9;
}
</style>