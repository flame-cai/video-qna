<!-- eslint-disable vue/multi-word-component-names -->
<script setup>
import { ref, watch } from 'vue'
import PlayerComponentLoader from './PlayerComponentLoader.vue'
import MediaPlayerComponent from './MediaPlayerComponent.vue'
import AnswerForm from './AnswerForm.vue'

const props = defineProps(['qna_promise', 'url'])
const qna = ref()
const url = ref(props.url)
const isDataLoaded = ref(false)
const isVideoPlaying = ref(false)
const reload_key = ref(0)
const currentChapterNumber = ref(0)

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

if (props.qna_promise) {
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
      localStorage.setItem('qna', result.data)
      isDataLoaded.value = true
    })
    .catch((e) => {
      console.error(e)
      alert('Something went wrong...')
      window.location.reload()
    })
} else {
  url.value = localStorage.getItem('url')
  qna.value = localStorage.getItem('qna')
}

watch(reload_key, () => {
  if (reload_key.value > 10) {
    window.location.reload()
  }
})
</script>

<template>
  <div v-if="isDataLoaded" class="player-component">
    <MediaPlayerComponent
      :qna="qna"
      :reload_key="reload_key"
      :url="url"
      @update-chapter="(n) => (currentChapterNumber = n)"
      @reload="reload_key++"
      @update-video-playing="(condition) => (isVideoPlaying = condition)"
    />
    <AnswerForm
      v-if="currentChapterNumber > 0"
      :qna="qna"
      :currentChapterNumber="currentChapterNumber"
      :isVideoPlaying="isVideoPlaying"
    />
  </div>
  <PlayerComponentLoader v-else />
</template>

<style>
div.player-component {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}
</style>
