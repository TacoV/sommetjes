<template>
  <div class="container" :class="feedbackClass">
    <!-- Tracking Row -->
    <div class="tracking">
      <span v-for="(item, index) in history" :key="index" :class="['mark', item ? 'correct' : 'wrong']">
        {{ item ? '✔' : '✘' }}
      </span>
    </div>

    <!-- Question -->
    <div class="question">
      {{ a }} + {{ b }} = ?
    </div>

    <!-- Answer Buttons -->
    <div class="answers">
      <button v-for="option in options" :key="option" @click="checkAnswer(option)" class="answer-button"
        :class="buttonClass(option)" :disabled="locked">
        {{ option }}
      </button>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { ref, computed } from 'vue'
import confetti from 'canvas-confetti'

const a = ref(0)
const b = ref(0)
const correctAnswer = ref(0)

const options = ref<number[]>([])
const history = ref<boolean[]>([])

const locked = ref(false)
const lastSelected = ref<number | null>(null)
const wasCorrect = ref<boolean | null>(null)

function randomInt(max: number): number {
  return Math.floor(Math.random() * max) + 1;
}

function generateQuestion() {
  a.value = 100
  b.value = 100
  while (a.value + b.value > 12) {
    a.value = randomInt(9)
    b.value = randomInt(9)
  }
  correctAnswer.value = a.value + b.value

  const answers = new Set<number>()
  answers.add(correctAnswer.value)

  while (answers.size < 4) {
    answers.add(randomInt(Math.max(correctAnswer.value + 2, 10)))
  }

  options.value = Array.from(answers).sort(() => Math.random() - 0.5)

  lastSelected.value = null
  wasCorrect.value = null
}

function celebrate() {
  confetti({
    particleCount: 80,
    spread: 70,
    origin: { y: 0.6 }
  })
}

function checkAnswer(selected: number) {
  if (locked.value) return

  locked.value = true
  lastSelected.value = selected

  const correct = selected === correctAnswer.value
  wasCorrect.value = correct

  history.value.unshift(correct)
  if (history.value.length > 10) history.value.pop()

  if (correct) {
    celebrate()
  }

  setTimeout(() => {
    locked.value = false
    generateQuestion()
  }, 1200)
}

const feedbackClass = computed(() => {
  if (wasCorrect.value === true) return 'correct-flash'
  if (wasCorrect.value === false) return 'wrong-flash'
  return ''
})

function buttonClass(option: number) {
  if (!locked.value) return ''

  if (option === correctAnswer.value) return 'highlight-correct'

  if (option === lastSelected.value && wasCorrect.value === false)
    return 'highlight-wrong'

  return ''
}

generateQuestion()
</script>

<style scoped>
.container {
  text-align: center;
  font-family: sans-serif;
  padding: 20px;
  transition: background-color 0.3s ease;
}

/* Screen flash feedback */
.correct-flash {
  background-color: #e6ffe6;
}

.wrong-flash {
  background-color: #ffe6e6;
}

/* Tracking row */
.tracking {
  font-size: 2rem;
  margin-bottom: 20px;
  min-height: 40px;
}

.mark {
  margin: 0 5px;
}

.correct {
  color: green;
}

.wrong {
  color: red;
}

/* Question */
.question {
  font-size: 4rem;
  margin: 30px 0;
}

/* Answers grid */
.answers {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 20px;
  max-width: 400px;
  margin: 0 auto;
}

/* Buttons */
.answer-button {
  font-size: 3rem;
  padding: 30px;
  border-radius: 20px;
  border: none;
  background-color: #f0f0f0;
  cursor: pointer;
  transition: transform 0.1s ease, background-color 0.3s ease;
}

.answer-button:active {
  transform: scale(0.95);
}

/* Correct highlight */
.highlight-correct {
  background-color: #4caf50 !important;
  color: white;
  transform: scale(1.1);
}

/* Wrong highlight */
.highlight-wrong {
  background-color: #f44336 !important;
  color: white;
  animation: shake 0.4s;
}

/* Shake animation */
@keyframes shake {
  0% {
    transform: translateX(0);
  }

  25% {
    transform: translateX(-6px);
  }

  50% {
    transform: translateX(6px);
  }

  75% {
    transform: translateX(-6px);
  }

  100% {
    transform: translateX(0);
  }
}
</style>