<script setup>
import { ref } from 'vue'
import LandingPageForm from './components/LandingPageForm.vue'
import Player from './components/player-components/Player.vue'

let progress = ref(0)
const url = ref()
const qna_promise = ref()

function progressNext() {
  progress.value += 1
}

if ('qna' in localStorage && 'url' in localStorage) {
  progressNext()
}

function qnaReceived(promise) {
  qna_promise.value = promise
  console.log(promise)
  progressNext()
}

function setUrl(url_string) {
  url.value = url_string
  localStorage.setItem('url', url_string)
}
</script>

<template>
  <header>
    <h1>Video Q&A Generator</h1>
  </header>
  <LandingPageForm v-if="progress === 0" @qna-received="qnaReceived" @set-url="setUrl" />
  <Player v-else-if="progress === 1" :qna_promise="qna_promise" :url="url" :qna="qna" />
</template>

<style scoped></style>
