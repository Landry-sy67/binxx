<script setup>
import { onBeforeUnmount, onMounted, ref, watch } from 'vue'

const props = defineProps({
  active: Boolean
})

const emit = defineEmits(['close'])

const petals = ref([])
const sparkles = ref([])

function burstPetals() {
  petals.value = Array.from({ length: 34 }, (_, index) => ({
    id: index,
    left: Math.random() * 100,
    delay: Math.random() * 0.5,
    duration: 3.2 + Math.random() * 2.2,
    rotate: Math.random() * 360,
    size: 10 + Math.random() * 8
  }))

  sparkles.value = Array.from({ length: 18 }, (_, index) => ({
    id: index + 100,
    left: Math.random() * 100,
    top: Math.random() * 100,
    delay: Math.random() * 0.7
  }))
}

const songUrl = '/Radiohead%20-%20Let%20Down%20[-Z_NvVMUcG8].mp3'
const isMuted = ref(false)
let audio = null

function startFinalAudio() {
  if (!audio) {
    audio = new Audio(songUrl)
    audio.loop = true
    audio.volume = 0.35
  }

  audio.play().catch(() => {})
  isMuted.value = false
}

function toggleMute() {
  if (!audio) {
    audio = new Audio(songUrl)
    audio.loop = true
    audio.volume = 0.35
    audio.play().catch(() => {})
  }

  audio.muted = !audio.muted
  isMuted.value = audio.muted
}

onMounted(() => {
  burstPetals()
  if (props.active) {
    startFinalAudio()
  }
})

watch(
  () => props.active,
  (active) => {
    if (active) {
      startFinalAudio()
    } else if (audio) {
      audio.muted = true
      isMuted.value = true
    }
  },
  { immediate: true }
)

onBeforeUnmount(() => {
  if (audio) {
    audio.pause()
    audio = null
  }
})
</script>

<template>
  <section class="final-page" @click="burstPetals">
    <div class="rose-layer" aria-hidden="true">
      <span class="floating-rose" style="left: 20%; top: 12%; font-size: 24px; animation-delay: 0.2s">🌹</span>
      <span class="floating-rose" style="left: 76%; top: 16%; font-size: 22px; animation-delay: 1.2s">🌹</span>
      <span class="floating-rose" style="left: 52%; top: 78%; font-size: 20px; animation-delay: 2s">🌹</span>
    </div>
    <div class="petal-layer" aria-hidden="true">
      <div
        v-for="petal in petals"
        :key="petal.id"
        class="petal"
        :style="{
          left: `${petal.left}%`,
          animationDelay: `${petal.delay}s`,
          animationDuration: `${petal.duration}s`,
          transform: `rotate(${petal.rotate}deg)`,
          width: `${petal.size}px`,
          height: `${petal.size * 1.4}px`
        }"
      />
      <div
        v-for="sparkle in sparkles"
        :key="sparkle.id"
        class="sparkle"
        :style="{
          left: `${sparkle.left}%`,
          top: `${sparkle.top}%`,
          animationDelay: `${sparkle.delay}s`
        }"
      />
    </div>
    <div class="final-card">
      <div class="final-orb"></div>
      <div class="card-top">
        <button class="back-btn" @click.stop="emit('close')">Back</button>
        <button class="primary-btn soundtrack-btn" @click="toggleMute">
          <span>{{ isMuted ? 'Unmute' : 'Mute' }}</span>
        </button>
      </div>
      <p class="eyebrow">A final note</p>
      <h2>I am G. L.</h2>
      <p class="final-message">
        And if love had a language, it would sound like this — soft, sincere, and forever glowing.
        I carry a quiet strength, a warm heart, and a deep love for beautiful moments that feel timeless.
      </p>
      <div class="final-signature">With love, G. L.</div>
    </div>
  </section>
</template>
