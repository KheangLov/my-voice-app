<template>
  <div>
    <button @click="startRecording" :disabled="isRecording" style="margin-right: 8px;">Start Recording</button>
    <button @click="stopRecording" :disabled="!isRecording">Stop Recording</button>

    <p><strong>Transcript:</strong> {{ transcript }}</p>

    <h3>Recorded Clips</h3>
    <ul>
      <li
        v-for="(clip, index) in audioClips"
        :key="index"
        style="margin-bottom: 1em; display: flex; flex-direction: column; justify-content: start; align-items: start;"
      >
        Clip {{ index + 1 }} —
        <!-- Replace plain audio with wavesurfer player -->
        <WaveSurferPlayer :src="clip.url" />
      </li>
    </ul>
  </div>
</template>

<script lang="ts" setup>
import { ref } from 'vue'
import Artyom from 'artyom.js'
import WaveSurferPlayer from './components/WaveSurferPlayer.vue'

interface AudioClip {
  blob: Blob
  url: string
}

const isRecording = ref(false)
const transcript = ref('')
const audioClips = ref<AudioClip[]>([])

let mediaRecorder: MediaRecorder | null = null
let audioChunks: BlobPart[] = []

// Setup Artyom
const artyom = new Artyom()

function startArtyom(): void {
  artyom
    .initialize({
      lang: 'en-GB',
      continuous: true,
      listen: true,
      debug: false,
      speed: 1,
    })
    .then(() => {
      artyom.say("I'm listening, please speak")
      artyom.redirectRecognizedTextOutput((text: string) => {
        transcript.value = text
      })
    })
}

function stopArtyom(): void {
  artyom.fatality().then(() => {
    console.log("Artyom stopped")
  })
}

async function startRecording(): Promise<void> {
  try {
    const stream = await navigator.mediaDevices.getUserMedia({ audio: true })
    mediaRecorder = new MediaRecorder(stream)
    audioChunks = []

    mediaRecorder.ondataavailable = (e: BlobEvent) => {
      audioChunks.push(e.data)
    }

    mediaRecorder.onstop = () => {
      const audioBlob = new Blob(audioChunks, { type: 'audio/webm' })
      const audioUrl = URL.createObjectURL(audioBlob)
      audioClips.value.push({ blob: audioBlob, url: audioUrl })
    }

    mediaRecorder.start()
    isRecording.value = true

    startArtyom()
  } catch (err) {
    console.error('Failed to start recording:', err)
  }
}

function stopRecording(): void {
  if (mediaRecorder && mediaRecorder.state !== 'inactive') {
    mediaRecorder.stop()
    isRecording.value = false

    stopArtyom()
  }
}

// function base64ToBlob(base64: string, mimeType: string): Blob {
//   const byteString = atob(base64.split(',')[1])
//   const ab = new ArrayBuffer(byteString.length)
//   const ia = new Uint8Array(ab)
//   for (let i = 0; i < byteString.length; i++) {
//     ia[i] = byteString.charCodeAt(i)
//   }
//   return new Blob([ab], { type: mimeType })
// }

// async function sendAudioToChatbotAPI(audioBlob: Blob): Promise<{ text: string; audioBlob: Blob }> {
//   const formData = new FormData()
//   formData.append('audio', audioBlob, 'user-audio.webm')

//   try {
//     const response = await fetch('https://your-chatbot-api.example.com/voice-chat', {
//       method: 'POST',
//       body: formData,
//     })

//     if (!response.ok) throw new Error('API error')

//     // Expect JSON { text: string, audioBase64: string }
//     const data = await response.json()

//     // Convert base64 audio back to Blob (assuming 'audio/webm')
//     const audioBlobBot = base64ToBlob(data.audioBase64, 'audio/webm')

//     return {
//       text: data.text,
//       audioBlob: audioBlobBot,
//     }
//   } catch (e) {
//     console.error('Chatbot API error:', e)
//     return {
//       text: 'Sorry, there was an error.',
//       audioBlob: new Blob(),
//     }
//   }
// }
</script>
