<script setup lang="ts">
import { ref, watch } from 'vue'

const props = defineProps(['qna', 'currentChapterNumber', 'isVideoPlaying'])

const submission = ref('')
const score = ref(0)

const answerIsCorrect = ref(false)
const answerIsWrong = ref(false)
const explanation = ref('')

watch(
  () => props.isVideoPlaying,
  () => {
    if (props.isVideoPlaying) {
      answerIsCorrect.value = false
      answerIsWrong.value = false
    }
  }
)

function submitAnswer() {
  fetch(`https://lacewing-relevant-mosquito.ngrok-free.app/evaluate-answer`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json' // Set the Content-Type header
    },
    body: JSON.stringify({
      question: props.qna[props.currentChapterNumber - 1][4],
      answer: props.qna[props.currentChapterNumber - 1][5],
      submission: submission.value
    })
  })
    .then((response) => response.json())
    .then((object) => {
      console.log(object)
      const data = object.data
      if (data.isCorrect) {
        answerIsCorrect.value = true
        score.value++
      } else {
        answerIsWrong.value = true
        explanation.value = data.explanation
      }
    })
}
</script>

<template>
  <div v-if="currentChapterNumber > 0">
    <form @submit.prevent="submitAnswer" id="answerForm">
      <label for="submission">Q. {{ props.qna[props.currentChapterNumber - 1][4] }}</label>
      <textarea
        v-model="submission"
        rows="4"
        :class="{ correct: answerIsCorrect, wrong: answerIsWrong }"
      ></textarea>
      <button type="submit">Submit Answer</button>
    </form>
    <div>
      <p v-if="answerIsWrong">{{ explanation }}</p>
      <p>Your Score: {{ score }}</p>
    </div>
  </div>
</template>
