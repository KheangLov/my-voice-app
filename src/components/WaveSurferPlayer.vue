<template>
  <div style="width: 100%; display: flex; flex-direction: column; align-items: start;">
    <div ref="waveform" style="width: 100%; height: 100px;"></div>
    <button @click="togglePlay" style="margin-top: 8px;">
      {{ isPlaying ? 'Pause' : 'Play' }}
    </button>
  </div>
</template>

<script lang="ts" setup>
import { ref, watch, onMounted, onBeforeUnmount } from 'vue'
import WaveSurfer from 'wavesurfer.js'

const props = defineProps<{ src: string }>()

let wavesurfer: WaveSurfer | null = null

const waveform = ref<HTMLDivElement | null>(null)
const isPlaying = ref(false)

function togglePlay() {
  if (!wavesurfer) return
  wavesurfer.playPause()
}

watch(
  () => props.src,
  (newSrc) => {
    if (wavesurfer && newSrc) {
      wavesurfer.load(newSrc)
    }
  }
)

onMounted(() => {
  if (!waveform.value) return

  wavesurfer = WaveSurfer.create({
    container: waveform.value,
    waveColor: '#a0a0a0',
    progressColor: '#4caf50',
    height: 80,
    responsive: true,
    normalize: true,
  })

  if (props.src) {
    wavesurfer.load(props.src)
  }

  wavesurfer.on('play', () => {
    isPlaying.value = true
  })

  wavesurfer.on('pause', () => {
    isPlaying.value = false
  })

  wavesurfer.on('finish', () => {
    isPlaying.value = false
  })
})

onBeforeUnmount(() => {
  if (wavesurfer) {
    wavesurfer.destroy()
  }
})
</script>

<style scoped>
button {
  cursor: pointer;
}
</style>
