<script setup>
import { ref } from 'vue'

const url = ref('')
const question_format = ref()

const emit = defineEmits(['qna-received', 'set-url', 'set-qf'])

function submitForm() {
  emit(
    'qna-received',
    fetch(`${import.meta.env.VITE_BACKEND_URL}/generate-video`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'ngrok-skip-browser-warning': 1
      },
      body: JSON.stringify({
        url: url.value,
        question_format: question_format.value
      })
    })
  )
  emit('set-url', url.value)
  emit('set-qf', question_format.value)
}
</script>

<template>
  <div class="landing-page-component">
    <form @submit.prevent="submitForm" id="videoForm">
      <input v-model="url" type="url" placeholder="Enter the url of the video" required />
      Question Format:
      <div>
        <label>MCQ</label
        ><input
          type="radio"
          name="question_format"
          v-model="question_format"
          id="question_format_mcq"
          value="mcq"
          required
        />
        <label>Subjective</label
        ><input
          type="radio"
          name="question_format"
          v-model="question_format"
          id="question_format_subjective"
          value="subjective"
          required
        />
      </div>
      <button type="submit">Get Questions</button>
    </form>
  </div>
</template>

<style scoped>
div.landing-page-component {
  flex-grow: 1;
  display: flex;
  justify-content: center;
  align-items: center;
}

form {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  width: 80vw;
  max-width: 540px;
}

input[type='url'] {
  width: 100%;
  border: 2.1px solid white;
  border-radius: 10px;
  padding: 0.5em;
  background: transparent;
  color: white;
  margin-bottom: 1em;
}

input[type='url']:focus {
  border-color: transparent;
  outline: orange solid;
}

button {
  /* width: 30%; */
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
