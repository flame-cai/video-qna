<script setup>
import { ref } from 'vue'
import LandingPageForm from './components/LandingPageForm.vue'
import QNAComponent from './components/qna-app-components/QNAComponent.vue'

let progress = ref(0)
const url = ref()
const question_format = ref()
const qna_promise = ref()

function progressNext() {
  progress.value += 1
}

if ('qna' in localStorage && 'url' in localStorage && 'question_format' in localStorage) {
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

function setQuestionFormat(qf) {
  question_format.value = qf
  localStorage.setItem('question_format', qf)
}
</script>

<template>
  <header>
    <h1>Video Q&A Generator</h1>
  </header>
  <LandingPageForm
    v-if="progress === 0"
    @qna-received="qnaReceived"
    @set-url="setUrl"
    @set-qf="setQuestionFormat"
  />
  <QNAComponent
    v-else-if="progress === 1"
    :qna_promise="qna_promise"
    :url="url"
    :qna="qna"
    :question_format="question_format"
  />
</template>

<style scoped></style>
