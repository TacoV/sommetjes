<template>
  <div class="container">
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
      <button v-for="option in options" :key="option" @click="checkAnswer(option)" class="answer-button">
        {{ option }}
      </button>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { ref } from 'vue'

const a = ref(0)
const b = ref(0)
const correctAnswer = ref(0)

const options = ref<number[]>([])
const history = ref<boolean[]>([]) // true = correct, false = wrong

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
}

function checkAnswer(selected: number) {
  const isCorrect = selected === correctAnswer.value

  history.value.unshift(isCorrect)
  if (history.value.length > 10) {
    history.value.pop()
  }

  generateQuestion()
}

generateQuestion()
</script>

<style scoped>
.container {
  text-align: center;
  font-family: sans-serif;
  padding: 20px;
}

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

.question {
  font-size: 4rem;
  margin: 30px 0;
}

.answers {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 20px;
  max-width: 400px;
  margin: 0 auto;
}

.answer-button {
  font-size: 3rem;
  padding: 30px;
  border-radius: 20px;
  border: none;
  background-color: #f0f0f0;
  cursor: pointer;
}

.answer-button:active {
  background-color: #ddd;
}
</style>