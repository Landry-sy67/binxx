<script setup>
import { onBeforeUnmount, onMounted, ref, watch, computed, nextTick } from 'vue'

const props = defineProps({
  active: Boolean
})

const emit = defineEmits(['close'])

const poemLines = [
  'In the quiet glow of your gaze, I found my tomorrow.',
  'You are the dream I never knew I was waiting for.',
  'Every portrait of you is a love letter to the future.',
  'Soft as moonlight, bright as a thousand stars.',
  'Your smile etches eternity into ordinary days.',
  'A future written in tender light and endless grace.',
  'You are the poetry my heart has been memorizing.',
  'With you, every moment feels like the first page of forever.',
  'Some souls just glow — you are the whole constellation.',
  'In a world of noise, you are my soft signal in the dark.',
  'You are not just my future. You are my always.'
]

const galleryImages = Array.from({ length: 11 }, (_, index) => ({
  id: index + 1,
  title: `Gigi ${index + 1}`,
  description: 'A tender frame from her soft, luminous story.',
  image: `/gigi${index + 1}.png`,
  poem: poemLines[index] || 'You are the dream I never knew I was waiting for.'
}))

const songUrl = '/frank%20ocean%20-%20american%20wedding%20[xQ6TZGc-IHU].mp3'
const isMuted = ref(false)

const currentSlide = ref(0)
const slideDirection = ref('next')
const mounted = ref(false)
const filmstripRef = ref(null)
const typedPoem = ref('')
const typingActive = ref(false)
let typeTimer = null

function startTyping() {
  clearInterval(typeTimer)
  typedPoem.value = ''
  typingActive.value = true
  const poem = galleryImages[currentSlide.value].poem
  if (!poem) { typingActive.value = false; return }
  let i = 0
  typeTimer = setInterval(() => {
    typedPoem.value += poem[i]
    i++
    if (i >= poem.length) {
      clearInterval(typeTimer)
      typingActive.value = false
    }
  }, 38)
}

watch(currentSlide, () => {
  setTimeout(startTyping, 420)
})

watch(mounted, (val) => {
  if (val) setTimeout(startTyping, 420)
})

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

function goToSlide(index) {
  if (index === currentSlide.value) return
  slideDirection.value = index > currentSlide.value ? 'next' : 'prev'
  currentSlide.value = index
  if (filmstripRef.value) {
    const thumb = filmstripRef.value.children[index]
    if (thumb) thumb.scrollIntoView({ behavior: 'smooth', inline: 'center', block: 'nearest' })
  }
}

function nextSlide() {
  if (currentSlide.value < galleryImages.length - 1) {
    goToSlide(currentSlide.value + 1)
  }
}

function prevSlide() {
  if (currentSlide.value > 0) {
    goToSlide(currentSlide.value - 1)
  }
}

function handleKeydown(e) {
  if (e.key === 'ArrowRight') nextSlide()
  if (e.key === 'ArrowLeft') prevSlide()
}

const slideImage = computed(() => galleryImages[currentSlide.value])

let audio = null

function startImageAudio() {
  if (!audio) {
    audio = new Audio(songUrl)
    audio.loop = true
    audio.volume = 0.35
  }

  audio.currentTime = 50
  audio.play().catch(() => {})
  isMuted.value = false
  nextTick(() => setupVisualizer())
}

function toggleMute() {
  if (!audio) {
    audio = new Audio(songUrl)
    audio.loop = true
    audio.volume = 0.35
    audio.play().catch(() => {})
    nextTick(() => setupVisualizer())
  }

  audio.muted = !audio.muted
  isMuted.value = audio.muted
}

const canvasRef = ref(null)
let audioCtx = null
let analyser = null
let dataArray = null
let animFrameId = null
let visualizerReady = false

function setupVisualizer() {
  if (visualizerReady || !audio || !canvasRef.value) return
  if (audio.readyState < 2) {
    audio.addEventListener('canplay', () => setupVisualizer(), { once: true })
    return
  }
  try {
    audioCtx = new (window.AudioContext || window.webkitAudioContext)()
    analyser = audioCtx.createAnalyser()
    analyser.fftSize = 128
    const source = audioCtx.createMediaElementSource(audio)
    source.connect(analyser)
    const bufferLength = analyser.frequencyBinCount
    dataArray = new Uint8Array(bufferLength)
    visualizerReady = true
    const canvas = canvasRef.value
    canvas.width = canvas.clientWidth * devicePixelRatio
    canvas.height = canvas.clientHeight * devicePixelRatio
    tickVisualizer()
  } catch (e) {
    visualizerReady = false
  }
}

function tickVisualizer() {
  if (!visualizerReady || !canvasRef.value) return
  animFrameId = requestAnimationFrame(tickVisualizer)
  analyser.getByteFrequencyData(dataArray)
  const canvas = canvasRef.value
  const ctx = canvas.getContext('2d')
  const w = canvas.width, h = canvas.height
  ctx.clearRect(0, 0, w, h)
  const len = dataArray.length
  const barW = (w / len) * 2.2
  let x = 0
  for (let i = 0; i < len; i++) {
    const bh = (dataArray[i] / 255) * h
    const grad = ctx.createLinearGradient(0, h, 0, h - bh)
    grad.addColorStop(0, 'rgba(255, 118, 194, 0.25)')
    grad.addColorStop(1, 'rgba(121, 92, 255, 0.45)')
    ctx.fillStyle = grad
    ctx.fillRect(x, h - bh, Math.max(barW - 1, 1), bh)
    x += barW
  }
}

function cleanupVisualizer() {
  if (animFrameId) cancelAnimationFrame(animFrameId)
  animFrameId = null
  if (audioCtx) audioCtx.close().catch(() => {})
  audioCtx = null
  analyser = null
  dataArray = null
  visualizerReady = false
}

onMounted(() => {
  burstPetals()
  mounted.value = true
  document.addEventListener('keydown', handleKeydown)
})

onBeforeUnmount(() => {
  clearInterval(typeTimer)
  cleanupVisualizer()
  document.removeEventListener('keydown', handleKeydown)
  if (audio) {
    audio.pause()
    audio = null
  }
})

watch(
  () => props.active,
  (active) => {
    if (active) {
      startImageAudio()
    } else if (audio) {
      audio.muted = true
      isMuted.value = true
    }
  },
  { immediate: true }
)
</script>

<template>
  <section class="image-page" @click="burstPetals">
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
    <div class="rose-layer" aria-hidden="true">
      <span class="floating-rose" style="left: 8%; top: 10%; font-size: 22px; animation-delay: 0.1s">🌹</span>
      <span class="floating-rose" style="left: 84%; top: 70%; font-size: 18px; animation-delay: 1.6s">🌹</span>
    </div>
    <div class="card-top">
      <button class="back-btn" @click.stop="emit('close')">Back</button>
      <div class="card-top-right">
        <span class="slideshow-badge">{{ slideImage.title }} · {{ String(currentSlide + 1).padStart(2, '0') }} / {{ String(galleryImages.length).padStart(2, '0') }}</span>
        <button class="primary-btn soundtrack-btn" @click="toggleMute">
          <span>{{ isMuted ? 'Unmute' : 'Mute' }}</span>
        </button>
      </div>
    </div>

    <div class="slideshow-main">
      <button class="slideshow-arrow slideshow-arrow--prev" @click="prevSlide" :disabled="currentSlide === 0">
        <svg width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M15 18l-6-6 6-6"/></svg>
      </button>

      <div class="slideshow-stage">
        <Transition :name="'slide-' + slideDirection" mode="out-in">
          <div v-if="mounted" :key="currentSlide" class="slideshow-slide">
            <div class="slideshow-image-wrap">
              <img :src="slideImage.image" :alt="slideImage.title" />
              <div class="slideshow-image-glow"></div>
            </div>
            <div class="slideshow-copy">
              <div class="slideshow-poem-wrap">
                <p class="slideshow-poem"><span class="typed-text">{{ typedPoem }}<span v-if="typingActive" class="typing-cursor">|</span></span></p>
              </div>
              <canvas ref="canvasRef" class="audio-visualizer"></canvas>
            </div>
          </div>
        </Transition>
      </div>

      <button class="slideshow-arrow slideshow-arrow--next" @click="nextSlide" :disabled="currentSlide === galleryImages.length - 1">
        <svg width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M9 18l6-6-6-6"/></svg>
      </button>
    </div>

    <div class="filmstrip">
      <div ref="filmstripRef" class="filmstrip-track">
        <button
          v-for="(item, index) in galleryImages"
          :key="item.id"
          class="filmstrip-thumb"
          :class="{ active: index === currentSlide }"
          @click="goToSlide(index)"
        >
          <img :src="item.image" :alt="item.title" loading="lazy" />
        </button>
      </div>
    </div>

  </section>
</template>
