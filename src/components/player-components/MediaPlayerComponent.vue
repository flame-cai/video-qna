<script setup>
import 'vidstack/bundle'
import { onMounted, ref, useTemplateRef } from 'vue'

const props = defineProps(['qna', 'reload_key', 'url'])
const emit = defineEmits(['update-chapter', 'reload', 'update-video-playing'])

const playerRef = useTemplateRef('player')
const isAd = ref(false)

let currentChapterNumber = 0
let currentChapterStartingTimestamp = 0
let currentChapterEndingTimestamp = 0

function convertToSeconds(timeStr) {
  const [hours, minutes, seconds] = timeStr.split(':').map(Number)
  return hours * 3600 + minutes * 60 + seconds
}

onMounted(() => {
  playerRef.value.subscribe(({ duration }) => {
    if (Math.abs(duration - convertToSeconds(props.qna[props.qna.length - 1][3])) > 10) {
      isAd.value = true
      console.log('duration', duration)
      emit('reload')
    } else {
      isAd.value = false
    }
  })

  playerRef.value.subscribe(({ currentTime }) => {
    if (
      !(
        currentTime >= currentChapterStartingTimestamp &&
        currentTime < currentChapterEndingTimestamp
      )
    ) {
      for (const chapter in props.qna) {
        const startingTimestamp = convertToSeconds(props.qna[chapter][2])
        const endingTimestamp = convertToSeconds(props.qna[chapter][3])
        if (currentTime >= startingTimestamp && currentTime < endingTimestamp) {
          currentChapterNumber = chapter
          emit('update-chapter', currentChapterNumber)
          currentChapterStartingTimestamp = startingTimestamp
          currentChapterEndingTimestamp = endingTimestamp
          console.log('current Chapter:', currentChapterNumber)
          console.log('currentChapterStartingTimestamp: ', currentChapterStartingTimestamp)
          console.log('currentChapterEndingTimestamp: ', currentChapterEndingTimestamp)
        }
      }
      if (currentTime >= convertToSeconds(props.qna[props.qna.length - 1][3])) {
        currentChapterNumber = props.qna.length
        emit('update-chapter', currentChapterNumber)
      }
      playerRef.value.pause()
    }
  })

  playerRef.value.subscribe(({ playing }) => {
    console.log('fired playing')
    console.log(playing)
    if (playing) {
      console.log('Playing')
      emit('update-video-playing', true)
    } else {
      console.log('Paused')
      emit('update-video-playing', false)
    }
  })
})
</script>

<template>
  <media-player ref="player" title="Demo" :src="url" class="video">
    <media-provider></media-provider>
    <media-video-layout></media-video-layout>
  </media-player>
</template>

<style>
media-player.video {
  max-width: 540px;
}
</style>
