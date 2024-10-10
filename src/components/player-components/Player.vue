<!-- eslint-disable vue/multi-word-component-names -->
<script setup>
import 'vidstack/bundle'
import { useTemplateRef, watch } from 'vue'
import { ref } from 'vue'
import PlayerComponentLoader from './PlayerComponentLoader.vue'
import AnswerForm from './AnswerForm.vue'

const props = defineProps(['qna_promise', 'url'])

const playerRef = useTemplateRef('player')
// const submission = ref('')
const qna = ref()
const isDataLoaded = ref(false)
const isVideoPlaying = ref(false)
let isAd = ref(false)

// const score = ref(0)
// const answerIsCorrect = ref(false)
// const answerIsWrong = ref(false)
// const explanation = ref('')

const currentChapterNumber = ref(0)
let currentChapterStartingTimestamp = 0
let currentChapterEndingTimestamp = 0

async function checkTaskStatus(taskId) {
  let response
  let result
  do {
    try {
      response = await fetch(`${import.meta.env.VITE_BACKEND_URL}/generate-video/${taskId}`, {
        headers: {
          'ngrok-skip-browser-warning': 1
        }
      })

      if (!response.ok) {
        throw new Error(`Error: ${response.status}`) // Trigger error handling
      }

      result = await response.json()
      console.log(result.status)

      if (result.status === 'completed') {
        console.log('Task Result:', result.data) // Handle the result here
        return result
      }

      // Wait 2 seconds before checking again
      await new Promise((resolve) => setTimeout(resolve, 2000))
    } catch (e) {
      console.error(e)
      alert('Something went wrong...')
      window.location.reload() // Reload the page on error
      break
    }
  } while (result.status !== 'completed')
}

props.qna_promise
  .then((response) => {
    console.log('response:', response)
    return response.json()
  })
  .then((object) => {
    console.log('object:', object)
    const taskId = object.taskId
    return checkTaskStatus(taskId)
  })
  .then((result) => {
    console.log('result:', result)
    qna.value = result.data
    currentChapterEndingTimestamp = convertToSeconds(qna.value[0][3])
    isDataLoaded.value = true
  })
  .catch((e) => {
    console.error(e)
    alert('Something went wrong...')
    window.location.reload()
  })

function convertToSeconds(timeStr) {
  const [hours, minutes, seconds] = timeStr.split(':').map(Number)
  return hours * 3600 + minutes * 60 + seconds
}

watch(isDataLoaded, () => {
  playerRef.value.subscribe(({ duration }) => {
    if (Math.abs(duration - convertToSeconds(qna.value[qna.value.length - 1][3])) > 10) {
      isAd.value = true
      console.log('duration', duration)
      console.log('last chapter ending')
    } else {
      isAd.value = false
    }
  })
  playerRef.value.subscribe(({ currentTime }) => {
    if (isAd.value) {
      //Ad is active as a video
      console.log('this is an ad')
      return
    }
    if (
      !(
        currentTime >= currentChapterStartingTimestamp &&
        currentTime < currentChapterEndingTimestamp
      )
    ) {
      for (const chapter in qna.value) {
        const startingTimestamp = convertToSeconds(qna.value[chapter][2])
        const endingTimestamp = convertToSeconds(qna.value[chapter][3])
        if (currentTime >= startingTimestamp && currentTime < endingTimestamp) {
          currentChapterNumber.value = chapter
          currentChapterStartingTimestamp = startingTimestamp
          currentChapterEndingTimestamp = endingTimestamp
          console.log('current Chapter:', currentChapterNumber)
          console.log('currentChapterStartingTimestamp: ', currentChapterStartingTimestamp)
          console.log('currentChapterEndingTimestamp: ', currentChapterEndingTimestamp)
        }
      }
      if (currentTime >= convertToSeconds(qna.value[qna.value.length - 1][3])) {
        currentChapterNumber.value = qna.value.length
      }
      playerRef.value.pause()
    }
  })
  playerRef.value.subscribe(({ playing }) => {
    console.log('fired playing')
    console.log(playing)
    if (playing) {
      isVideoPlaying.value = true
      console.log('after', isVideoPlaying)
    } else {
      console.log('huh')
      isVideoPlaying.value = false
    }
  })
})

// function submitAnswer() {
//   fetch(`http://localhost:5000/evaluate-answer`, {
//     method: 'POST',
//     headers: {
//       'Content-Type': 'application/json' // Set the Content-Type header
//     },
//     body: JSON.stringify({
//       question: qna.value[currentChapterNumber.value - 1][4],
//       answer: qna.value[currentChapterNumber.value - 1][5],
//       submission: submission.value
//     })
//   })
//     .then((response) => response.json())
//     .then((object) => {
//       console.log(object)
//       const data = object.data
//       if (data.isCorrect) {
//         answerIsCorrect.value = true
//         score.value++
//       } else {
//         answerIsWrong.value = true
//         explanation.value = data.explanation
//       }
//     })
// }
</script>

<template>
  <div v-show="isDataLoaded" class="player-component">
    <media-player ref="player" title="Demo" :src="url" class="video">
      <media-provider></media-provider>
      <media-video-layout></media-video-layout>
    </media-player>
    <!-- <div v-if="currentChapterNumber > 0">
      <form @submit.prevent="submitAnswer" id="answerForm">
        <label for="submission">Q. {{ qna[currentChapterNumber - 1][4] }}</label>
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
    </div> -->
    <AnswerForm
      v-if="currentChapterNumber > 0"
      :qna="qna"
      :currentChapterNumber="currentChapterNumber"
      :isVideoPlaying="isVideoPlaying"
    />
  </div>
  <PlayerComponentLoader v-if="!isDataLoaded" />
</template>

<style>
div.player-component {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

media-player.video {
  max-width: 540px;
}

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
