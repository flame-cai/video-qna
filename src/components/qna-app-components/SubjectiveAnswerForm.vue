<script setup>
import { ref, watch, computed } from 'vue'
import { useSpeechSynthesis } from '@vueuse/core';

import VolumeIcon from './VolumeIcon.vue';

const props = defineProps(['qna', 'currentChapterNumber', 'isVideoPlaying', 'isSubmitted', 'isCorrect', 'score'])
const emit = defineEmits(['submit-answer',])

const submission = ref('')

const answerIsCorrect = computed(() => {
  if (props.isCorrect) return true;
  else return false
})
const answerIsWrong = computed(() => {
  if (props.isSubmitted && !props.isCorrect) return true;
  else return false;
})
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

const speech = useSpeechSynthesis(props.qna[props.currentChapterNumber - 1]['chapter_question'], {
  rate: 0.75,
})

function submitAnswer() {
  fetch(`${import.meta.env.VITE_BACKEND_URL}/evaluate-answer`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json' // Set the Content-Type header
    },
    body: JSON.stringify({
      question: props.qna[props.currentChapterNumber - 1]['chapter_question'],
      answer: props.qna[props.currentChapterNumber - 1]['chapter_answer'],
      submission: submission.value
    })
  })
    .then((response) => response.json())
    .then((object) => {
      console.log(object)
      const data = object.data
      if (data.isCorrect) {
        emit('submit-answer', true);
      } else {
        emit('submit-answer', false);
        explanation.value = data.explanation;
      }
    })
}

function play() {
  if (speech.status.value === 'pause') {
    window.speechSynthesis.resume();
  } else {
    speech.stop();
    speech.speak();
  }
}

</script>

<template>
  <div v-if="currentChapterNumber > 0">
    <form @submit.prevent="submitAnswer" id="answerForm">
      <label for="submission">Q. {{ props.qna[props.currentChapterNumber - 1]['chapter_question'] }} <button
          @click.prevent="play">
          <VolumeIcon />
        </button></label>
      <textarea v-model="submission" rows="4" :class="{ correct: answerIsCorrect, wrong: answerIsWrong }"></textarea>
      <button type="submit" :disabled="props.isSubmitted">Submit Answer</button>
    </form>
    <div>
      <p v-if="answerIsWrong">{{ explanation }}</p>
      <p>Your Score: {{ props.score }}</p>
    </div>
  </div>
</template>

<style>
#answerForm {
  display: flex;
  flex-direction: column;
  justify-content: center;
}

#answerForm * {
  margin-bottom: 0.5em;
}

#answerForm textarea {
  border: 2.1px solid white;
  padding: 0.5em;
  background-color: transparent;
  color: white;
  border-radius: 10px;
}

#answerForm textarea.correct {
  border: 2.1px solid green;
}

#answerForm textarea.wrong {
  border: 2.1px solid red;
}

#answerForm textarea:focus {
  outline: orange solid;
  border-color: transparent;
}

button {
  border: 1px;
  border: 2.1px solid white;
  border-radius: 10px;
  padding: 0.5em;
  margin: auto;
}

button:hover {
  background-color: orange;
  border-color: orange;
  color: white;
}
</style>
