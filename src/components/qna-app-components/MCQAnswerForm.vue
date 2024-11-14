<script setup>
import { onMounted, ref, computed } from 'vue'
import { useSpeechSynthesis } from '@vueuse/core'

import VolumeIcon from './VolumeIcon.vue';

const props = defineProps(['qna', 'currentChapterNumber', 'isVideoPlaying', 'isSubmitted', 'isCorrect', 'score'])
const emit = defineEmits(['submit-answer',])

const selectedOption = ref('')

const answerIsCorrect = computed(() => {
  if (props.isCorrect) return true;
  else return false
})
const answerIsWrong = computed(() => {
  if (props.isSubmitted && !props.isCorrect) return true;
  else return false;
})

/**
 * This function submits the form
 */
function submitAnswer() {
  if (
    +selectedOption.value === props.qna[props.currentChapterNumber - 1]['correct_option_number']
  ) {
    emit('submit-answer', true)
  } else {
    emit('submit-answer', false)
  }
}

const TTStext = ref(
  props.qna[props.currentChapterNumber - 1]['question'] +
  '. Option A. ' +
  props.qna[props.currentChapterNumber - 1]['options'][0]['text'] +
  '. Option B. ' +
  props.qna[props.currentChapterNumber - 1]['options'][1]['text'] +
  '. Option C. ' +
  props.qna[props.currentChapterNumber - 1]['options'][2]['text'] +
  '. Option D. ' +
  props.qna[props.currentChapterNumber - 1]['options'][3]['text']
)
const speech = useSpeechSynthesis(TTStext, {
  rate: 0.75
})

let synth

onMounted(() => {
  if (speech.isSupported.value) {
    setTimeout(() => {
      synth = window.speechSynthesis
    })
  }
})

function play() {
  if (speech.status.value === 'pause') {
    console.log('resume')
    window.speechSynthesis.resume()
  } else {
    speech.stop()
    speech.speak()
  }
}
</script>

<template>
  <form class="mcq-form" @submit.prevent="submitAnswer" :class="{ correct: answerIsCorrect, wrong: answerIsWrong }">
    <p v-if="answerIsCorrect">Correct Answer</p>
    <p v-if="answerIsWrong">Wrong Answer</p>
    <label for="">Q. {{ props.qna[props.currentChapterNumber - 1]['question'] }} <button @click.prevent="play">
        <VolumeIcon />
      </button></label>
    <span><input v-model="selectedOption" type="radio" name="options" id="option_1" value="1" /><label for="option_1">{{
      props.qna[props.currentChapterNumber - 1]['options'][0]['text'] }}</label></span>
    <span><input v-model="selectedOption" type="radio" name="options" id="option_2" value="2" /><label for="option_2">{{
      props.qna[props.currentChapterNumber - 1]['options'][1]['text'] }}</label></span>
    <span><input v-model="selectedOption" type="radio" name="options" id="option_3" value="3" /><label for="option_3">{{
      props.qna[props.currentChapterNumber - 1]['options'][2]['text'] }}</label></span>
    <span><input v-model="selectedOption" type="radio" name="options" id="option_4" value="4" /><label for="option_4">{{
      props.qna[props.currentChapterNumber - 1]['options'][3]['text'] }}</label></span>
    <button type="submit" :disabled="props.isSubmitted">Submit</button>
  </form>
  <p>Your score: {{ props.score }}</p>
</template>

<style>
.mcq-form {
  display: flex;
  padding: 1em;
  flex-direction: column;
  margin: 1em;
  border: 2.1px transparent;
  border-radius: 10px;
}

.mcq-form.correct {
  border: 2.1px solid green;
}

.mcq-form.wrong {
  border: 2.1px solid red;
}
</style>
