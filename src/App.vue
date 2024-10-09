<script setup>
import { ref } from 'vue'
import LandingPageForm from './components/LandingPageForm.vue'
import Player from './components/player-components/Player.vue'

let progress = ref(0)
const url = ref()
// url.value = 'https://www.youtube.com/watch?v=l9AzO1FMgM8&pp=ygUNZmlyZXNoaXAgamF2YQ%3D%3D'
const qna_promise = ref()
// const qna_promise = ref(
//   fetch(
//     `https://edfd-49-248-175-215.ngrok-free.app/generate-video?url=${encodeURIComponent(url.value)}`
//   )
// )

function progressNext() {
  progress.value += 1
}

function qnaReceived(promise) {
  qna_promise.value = promise
  console.log(promise)
  progressNext()
}

function setUrl(url_string) {
  url.value = url_string
}
</script>

<template>
  <header>
    <h1>Video Q&A Generator</h1>
  </header>
  <LandingPageForm v-if="progress === 0" @qna-received="qnaReceived" @set-url="setUrl" />
  <Player v-else-if="progress === 1" :qna_promise="qna_promise" :url="url" />
</template>

<style scoped></style>
