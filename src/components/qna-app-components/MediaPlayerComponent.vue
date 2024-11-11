<script setup>
import 'vidstack/bundle'
import { onMounted, ref, useTemplateRef } from 'vue'

const props = defineProps(['qna', 'reload_key', 'url', 'duration'])
const emit = defineEmits(['update-chapter', 'reload', 'update-video-playing'])

const playerRef = useTemplateRef('player')
const isAd = ref(false)

let currentChapterNumber = 0
let currentChapterStartingTimestamp = 0
let currentChapterEndingTimestamp = 0

function convertToSeconds(timeStr) {
  console.log('timeStr:', timeStr)
  const parts = timeStr.split(':').map(Number)

  if (parts.length === 2) {
    // If only minutes and seconds are provided (mm:ss)
    const [minutes, seconds] = parts
    return minutes * 60 + seconds
  } else if (parts.length === 3) {
    // If hours, minutes, and seconds are provided (hh:mm:ss)
    const [hours, minutes, seconds] = parts
    return hours * 3600 + minutes * 60 + seconds
  } else {
    throw new Error('Invalid time format')
  }
}

onMounted(() => {
  playerRef.value.subscribe(({ duration }) => {
    console.log('duration:', props.duration)
    if (Math.abs(duration - convertToSeconds(props.duration)) > 1) {
      isAd.value = true
      console.log('duration', props.duration, convertToSeconds(props.duration))
      console.log('reload key:', props.reload_key)
      emit('reload')
    } else {
      isAd.value = false
    }
  })

  playerRef.value.subscribe(({ currentTime }) => {
    console.log(currentTime)
    if (isAd.value) {
      console.log('reload key:', props.reload_key)
      emit('reload')
    }

    if (
      !(
        currentTime >= currentChapterStartingTimestamp &&
        currentTime < currentChapterEndingTimestamp
      )
    ) {
      for (const chapter in props.qna) {
        const startingTimestamp = convertToSeconds(props.qna[chapter]['chapter_start_timestamp'])
        const endingTimestamp = convertToSeconds(props.qna[chapter]['chapter_end_timestamp'])
        console.log('starting and ending timestamps', startingTimestamp, endingTimestamp)
        console.log(currentTime)
        if (currentTime >= startingTimestamp && currentTime < endingTimestamp) {
          console.log('chapter', chapter)
          currentChapterNumber = chapter
          emit('update-chapter', currentChapterNumber)
          currentChapterStartingTimestamp = startingTimestamp
          currentChapterEndingTimestamp = endingTimestamp
          console.log('current Chapter:', currentChapterNumber)
          console.log('currentChapterStartingTimestamp: ', currentChapterStartingTimestamp)
          console.log('currentChapterEndingTimestamp: ', currentChapterEndingTimestamp)
        }
      }
      if (
        currentTime >= convertToSeconds(props.qna[props.qna.length - 1]['chapter_end_timestamp'])
      ) {
        currentChapterNumber = props.qna.length
        emit('update-chapter', currentChapterNumber)
      }
      playerRef.value.pause()
      if (document.fullscreenElement) {
        document.exitFullscreen()
      }

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
  <media-player ref="player" :src="url" class="video">
    <media-provider></media-provider>
    <media-video-layout></media-video-layout>
  </media-player>
</template>

<style>
media-player.video {
  max-width: 540px;
}
</style>
