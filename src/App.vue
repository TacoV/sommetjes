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
      <button v-for="(option, i) in options" :key="i" @click="checkAnswer(option)" class="answer-button"
        :class="buttonClass(option)" :disabled="locked">
        {{ option }}
      </button>
    </div>

    <div class="stats">
      Level: {{ level }}
    </div>
  </div>
</template>

<script lang="ts" setup>
import { ref, computed } from 'vue'
import confetti from 'canvas-confetti'

const level = ref(1)

const streak = ref(0)

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
  do {
    a.value = randomInt(level.value + 2)
    b.value = randomInt(level.value + 2)
  } while (a.value + b.value > level.value + 7)

  correctAnswer.value = a.value + b.value

  const answers = new Set<number>()
  answers.add(correctAnswer.value)

  const highest = Math.max(9, Math.min(level.value + 7, 2 * level.value + 4))
  while (answers.size < 9) {
    answers.add(randomInt(highest))
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

  streak.value = correct ? streak.value + 1 : 0
  if (streak.value > 5) {
    level.value++
    history.value = []
    streak.value = 0
  }

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
  height: 100%;
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
  min-height: 48px;
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

.stats {
  color: #999;
  margin-top: 2em;
  font-style: italic;
}

/* Answers grid */
.answers {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
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
  box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.2), 0 6px 20px 0 rgba(0, 0, 0, 0.19);
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