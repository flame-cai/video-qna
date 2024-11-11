<script setup>
import { onMounted, ref, watch } from 'vue'
import { useSpeechSynthesis } from '@vueuse/core'

const props = defineProps(['qna', 'currentChapterNumber', 'isVideoPlaying'])

const score = ref(0)
const selectedOption = ref('')
const answerIsCorrect = ref(false)
const answerIsWrong = ref(false)

watch(
  () => props.isVideoPlaying,
  () => {
    if (props.isVideoPlaying) {
      answerIsCorrect.value = false
      answerIsWrong.value = false
    }
  }
)

/**
 * This function submits the form
 */
function submitAnswer() {
  if (
    +selectedOption.value === props.qna[props.currentChapterNumber - 1]['correct_option_number']
  ) {
    score.value += 1
    answerIsCorrect.value = true
    answerIsWrong.value = false
  } else {
    answerIsWrong.value = true
    answerIsCorrect.value = false
  }
}

const voice = ref()
const TTStext = ref(
  props.qna[props.currentChapterNumber - 1]['question'] +
    '. .' +
    'A. ' +
    props.qna[props.currentChapterNumber - 1]['options'][0]['text'] +
    '. .' +
    'B. ' +
    props.qna[props.currentChapterNumber - 1]['options'][1]['text'] +
    '. .' +
    'C. ' +
    props.qna[props.currentChapterNumber - 1]['options'][2]['text'] +
    '. .' +
    'D. ' +
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
    speech.speak()
  }
}
</script>

<template>
  <form class="mcq-form" @submit.prevent="submitAnswer">
    <p v-if="answerIsCorrect">Correct Answer</p>
    <p v-if="answerIsWrong">Wrong Answer</p>
    <label for="">Q. {{ props.qna[props.currentChapterNumber - 1]['question'] }}</label>
    <span
      ><input v-model="selectedOption" type="radio" name="options" id="option_1" value="1" /><label
        for="option_1"
        >{{ props.qna[props.currentChapterNumber - 1]['options'][0]['text'] }}</label
      ></span
    >
    <span
      ><input v-model="selectedOption" type="radio" name="options" id="option_2" value="2" /><label
        for="option_2"
        >{{ props.qna[props.currentChapterNumber - 1]['options'][1]['text'] }}</label
      ></span
    >
    <span
      ><input v-model="selectedOption" type="radio" name="options" id="option_3" value="3" /><label
        for="option_3"
        >{{ props.qna[props.currentChapterNumber - 1]['options'][2]['text'] }}</label
      ></span
    >
    <span
      ><input v-model="selectedOption" type="radio" name="options" id="option_4" value="4" /><label
        for="option_4"
        >{{ props.qna[props.currentChapterNumber - 1]['options'][3]['text'] }}</label
      ></span
    >
    <button type="submit">Submit</button>
  </form>
</template>

<style>
.mcq-form {
  display: flex;
  flex-direction: column;
  margin: 1em;
}
</style>
