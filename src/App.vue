<script setup>
import { computed, nextTick, onBeforeUnmount, ref } from 'vue'
import confetti from 'canvas-confetti'
import { Volume2, VolumeX } from 'lucide-vue-next'
import { birthday as birthdayData } from './data/birthday'
import LaunchGate from './components/LaunchGate.vue'
import AmbientLab from './components/AmbientLab.vue'
import HeroStage from './components/HeroStage.vue'
import StorySections from './components/StorySections.vue'
import CandleFinale from './components/CandleFinale.vue'
import PhotoModal from './components/PhotoModal.vue'

const phase = ref('gate')
const soundOn = ref(true)
const activePhoto = ref(null)
const collected = ref([])
const toast = ref('')
let audioContext
let toastTimer

const birthday = computed(() => ({
  ...birthdayData,
  photos: birthdayData.photos.map((photo) => ({
    ...photo,
    src: `${import.meta.env.BASE_URL}${photo.src}`,
  })),
  finalePhoto: {
    ...birthdayData.finalePhoto,
    src: `${import.meta.env.BASE_URL}${birthdayData.finalePhoto.src}`,
  },
}))

function ensureAudio() {
  if (!audioContext) {
    audioContext = new (window.AudioContext || window.webkitAudioContext)()
  }
  if (audioContext.state === 'suspended') audioContext.resume()
  return audioContext
}

function playNotes(notes, type = 'sine') {
  if (!soundOn.value) return
  const context = ensureAudio()
  notes.forEach(({ frequency, delay, duration = 0.24 }) => {
    const oscillator = context.createOscillator()
    const gain = context.createGain()
    oscillator.type = type
    oscillator.frequency.value = frequency
    gain.gain.setValueAtTime(0.0001, context.currentTime + delay)
    gain.gain.exponentialRampToValueAtTime(0.12, context.currentTime + delay + 0.025)
    gain.gain.exponentialRampToValueAtTime(0.0001, context.currentTime + delay + duration)
    oscillator.connect(gain).connect(context.destination)
    oscillator.start(context.currentTime + delay)
    oscillator.stop(context.currentTime + delay + duration + 0.03)
  })
}

function startLaunch() {
  phase.value = 'scanning'
  playNotes([
    { frequency: 392, delay: 0 },
    { frequency: 523.25, delay: 0.16 },
    { frequency: 659.25, delay: 0.32 },
    { frequency: 783.99, delay: 0.48, duration: 0.5 },
  ], 'triangle')
}

async function completeLaunch() {
  phase.value = 'open'
  await nextTick()
  openingConfetti()
  playNotes([
    { frequency: 523.25, delay: 0 },
    { frequency: 659.25, delay: 0.08 },
    { frequency: 783.99, delay: 0.16 },
    { frequency: 1046.5, delay: 0.28, duration: 0.65 },
  ])
}

function openingConfetti() {
  const colors = ['#ff6072', '#b8f34a', '#a985ff', '#f1cd72', '#f7f2ea']
  confetti({ particleCount: 130, spread: 100, startVelocity: 48, origin: { y: 0.54 }, colors, scalar: 1.05 })
  window.setTimeout(() => {
    confetti({ particleCount: 80, angle: 60, spread: 65, origin: { x: 0, y: 0.72 }, colors })
    confetti({ particleCount: 80, angle: 120, spread: 65, origin: { x: 1, y: 0.72 }, colors })
  }, 360)
}

function celebrateFinale() {
  const end = Date.now() + 2200
  const colors = ['#ff6072', '#b8f34a', '#a985ff', '#f1cd72', '#ffffff']
  const frame = () => {
    confetti({ particleCount: 7, angle: 60, spread: 70, origin: { x: 0, y: 0.7 }, colors })
    confetti({ particleCount: 7, angle: 120, spread: 70, origin: { x: 1, y: 0.7 }, colors })
    if (Date.now() < end) requestAnimationFrame(frame)
  }
  frame()
  playNotes([
    { frequency: 392, delay: 0 },
    { frequency: 523.25, delay: 0.12 },
    { frequency: 659.25, delay: 0.24 },
    { frequency: 783.99, delay: 0.36 },
    { frequency: 1046.5, delay: 0.54, duration: 0.8 },
  ], 'triangle')
}

function collectNucleotide(letter) {
  if (collected.value.includes(letter)) return
  collected.value = [...collected.value, letter]
  playNotes([{ frequency: 420 + collected.value.length * 90, delay: 0 }], 'sine')
  showToast(`已发现生日基因 ${collected.value.length}/5`)
  if (collected.value.length === 5) {
    window.setTimeout(() => showToast('ACHIEVEMENT UNLOCKED · 发现生日基因'), 500)
    window.setTimeout(openingConfetti, 700)
  }
}

function showToast(message) {
  toast.value = message
  window.clearTimeout(toastTimer)
  toastTimer = window.setTimeout(() => { toast.value = '' }, 3000)
}

function toggleSound() {
  soundOn.value = !soundOn.value
  if (soundOn.value) playNotes([{ frequency: 659.25, delay: 0 }])
}

onBeforeUnmount(() => {
  window.clearTimeout(toastTimer)
  audioContext?.close()
})
</script>

<template>
  <main class="app-shell">
    <LaunchGate
      v-if="phase !== 'open'"
      :phase="phase"
      :birthday="birthday"
      @start="startLaunch"
      @complete="completeLaunch"
    />

    <template v-else>
      <AmbientLab :collected="collected" @collect="collectNucleotide" />

      <button
        class="icon-button sound-toggle"
        type="button"
        :aria-label="soundOn ? '关闭声音' : '打开声音'"
        :aria-pressed="soundOn"
        @click="toggleSound"
      >
        <Volume2 v-if="soundOn" :size="18" />
        <VolumeX v-else :size="18" />
      </button>

      <HeroStage :birthday="birthday" @open-photo="activePhoto = $event" />
      <StorySections :birthday="birthday" @open-photo="activePhoto = $event" @toast="showToast" />
      <CandleFinale :birthday="birthday" @celebrate="celebrateFinale" />

      <footer class="site-footer">
        <span>MADE WITH 22x SOUL</span>
        <span>生日快乐 · {{ birthday.name }}</span>
      </footer>
    </template>

    <Transition name="toast">
      <div v-if="toast" class="achievement-toast" role="status">{{ toast }}</div>
    </Transition>

    <PhotoModal :photo="activePhoto" @close="activePhoto = null" />
  </main>
</template>
