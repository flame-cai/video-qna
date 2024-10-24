<script setup>
import { ref, watch } from 'vue'

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
</script>

<template>
  <form>
    <label for="">Q. {{ props.qna[props.currentChapterNumber - 1]['chapter_question'] }}</label>
    <input v-model="selectedOption" type="radio" name="options" id="option_1" value="1" />{{
      props.qna[props.currentChapterNumber - 1]['options'][0]['text']
    }}
    <input v-model="selectedOption" type="radio" name="options" id="option_2" value="2" />{{
      props.qna[props.currentChapterNumber - 1]['options'][1]['text']
    }}
    <input v-model="selectedOption" type="radio" name="options" id="option_3" value="3" />{{
      props.qna[props.currentChapterNumber - 1]['options'][2]['text']
    }}
    <input v-model="selectedOption" type="radio" name="options" id="option_4" value="4" />{{
      props.qna[props.currentChapterNumber - 1]['options'][3]['text']
    }}
  </form>
</template>
