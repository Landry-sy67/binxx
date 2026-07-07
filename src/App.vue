<script setup>
import { computed, onMounted, onBeforeUnmount, provide, readonly, ref, shallowRef } from 'vue'
import AboutPage from './components/AboutPage.vue'
import ImagePage from './components/ImagePage.vue'
import FinalPage from './components/FinalPage.vue'

const petals = ref([])
const sparkles = ref([])

function burstPetals() {
  if (!audioStarted) initAudio(0)

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

const highlights = ['Kind', 'Dreamy', 'Future-minded', 'Tender friend']

const traits = [
  'She makes every conversation feel warm and effortless.',
  'Her kindness shows in the softest, most sincere details.',
  'She carries a quiet strength and a graceful, romantic energy.',
  'She turns ordinary moments into something beautiful and memorable.'
]

const favoriteThings = [
  'Starry skies, soft music, and midnight glow',
  'Sweet messages written like little love notes',
  'Gentle laughter and cozy talks that feel timeless',
  'Dreams of bright futures and beautiful little moments'
]

const galleryItems = Array.from({ length: 11 }, (_, index) => ({
  title: `Gigi ${index + 1}`,
  text: 'A luminous portrait from the Gigi collection.',
  image: `/gigi${index + 1}.png`
}))

const homeRoses = Array.from({ length: 10 }, (_, index) => ({
  id: index,
  left: 4 + Math.random() * 92,
  top: 8 + Math.random() * 78,
  delay: Math.random() * 5,
  size: 18 + Math.random() * 12
}))

const currentView = ref('home')

function openAbout() {
  navigateTo('about')
}

function openImages() {
  navigateTo('images')
}

function openFinal() {
  navigateTo('final')
}

const heroCardRef = ref(null)

function handleHeroGlow(e) {
  if (!heroCardRef.value) return
  const rect = heroCardRef.value.getBoundingClientRect()
  const x = ((e.clientX - rect.left) / rect.width) * 100
  const y = ((e.clientY - rect.top) / rect.height) * 100
  heroCardRef.value.style.setProperty('--mx', x + '%')
  heroCardRef.value.style.setProperty('--my', y + '%')
}

function handleHeroGlowLeave() {
  if (!heroCardRef.value) return
  heroCardRef.value.style.setProperty('--mx', '50%')
  heroCardRef.value.style.setProperty('--my', '50%')
}

// --- Page-specific songs with crossfade navigation ---
const trackForView = { home: 0, about: 1, images: 2, final: 3 }

const playlist = [
  { url: '/Frank Ocean - White Ferrari [Dlz_XHeUUis].mp3', name: 'White Ferrari' },
  { url: '/Golden%20Brown%20-%20The%20Stranglers%20(slowed%20+%20reverb)%20[GmxkNB-QihI].mp3', name: 'Golden Brown' },
  { url: '/frank%20ocean%20-%20american%20wedding%20[xQ6TZGc-IHU].mp3', name: 'American Wedding' },
  { url: '/Radiohead%20-%20Let%20Down%20[-Z_NvVMUcG8].mp3', name: 'Let Down' }
]

const currentTrackIndex = ref(0)
const isMuted = ref(false)
const audioElement = shallowRef(null)
let currentAudio = null
let crossfadeTimer = null
const CROSSFADE_SEC = 3
const BASE_VOLUME = 0.2
let audioStarted = false

function initAudio(trackIndex = 0) {
  if (audioStarted) return
  audioStarted = true
  currentTrackIndex.value = trackIndex
  currentAudio = new Audio(playlist[trackIndex].url)
  currentAudio.loop = true
  currentAudio.volume = BASE_VOLUME
  audioElement.value = currentAudio
  currentAudio.play().catch(() => {})
}

function toggleMute() {
  isMuted.value = !isMuted.value
  if (currentAudio) {
    currentAudio.muted = isMuted.value
    if (!isMuted.value) currentAudio.volume = BASE_VOLUME
  }
}

function crossfadeToTrack(index) {
  if (!currentAudio || index === currentTrackIndex.value) return

  const oldAudio = currentAudio
  const newAudio = new Audio(playlist[index].url)
  newAudio.loop = true
  newAudio.volume = 0
  if (isMuted.value) newAudio.muted = true

  newAudio.addEventListener('canplaythrough', () => {
    if (crossfadeTimer) {
      clearInterval(crossfadeTimer)
      crossfadeTimer = null
    }
    newAudio.play().then(() => {
      const steps = 30
      const interval = (CROSSFADE_SEC * 1000) / steps
      let step = 0

      crossfadeTimer = setInterval(() => {
        step++
        const progress = step / steps
        oldAudio.volume = Math.max(0, BASE_VOLUME * (1 - progress))
        newAudio.volume = Math.min(BASE_VOLUME, BASE_VOLUME * progress)

        if (step >= steps) {
          clearInterval(crossfadeTimer)
          crossfadeTimer = null
          oldAudio.pause()
          currentAudio = newAudio
          currentAudio.loop = true
          if (isMuted.value) currentAudio.muted = true
          audioElement.value = currentAudio
          currentTrackIndex.value = index
        }
      }, interval)
    }).catch(() => {})
  }, { once: true })
}

function navigateTo(view) {
  const targetTrack = trackForView[view]
  if (!audioStarted) {
    initAudio(targetTrack)
  } else if (targetTrack !== currentTrackIndex.value) {
    crossfadeToTrack(targetTrack)
  }
  currentView.value = view
}

function goHome() {
  navigateTo('home')
}

function cleanupAudio() {
  if (crossfadeTimer) {
    clearInterval(crossfadeTimer)
    crossfadeTimer = null
  }
  if (currentAudio) {
    currentAudio.pause()
    currentAudio = null
  }
}

provide('audioManager', {
  isMuted: readonly(isMuted),
  toggleMute,
  currentTrack: computed(() => playlist[currentTrackIndex.value]),
  audioElement: readonly(audioElement)
})

onMounted(() => {
  burstPetals()
})

onBeforeUnmount(() => {
  cleanupAudio()
})

</script>

<template>
  <main class="portfolio">
    <div class="orb-glow orb-glow--pink" aria-hidden="true"></div>
    <div class="orb-glow orb-glow--purple" aria-hidden="true"></div>
    <div class="orb-glow orb-glow--blue" aria-hidden="true"></div>
    <Transition name="page" mode="out-in">
    <div v-if="currentView === 'home'" key="home" class="page-transition-wrapper">
      <section ref="heroCardRef" class="hero-card" @click="burstPetals" @mousemove="handleHeroGlow" @mouseleave="handleHeroGlowLeave">
        <div class="hero-glow-follow" aria-hidden="true"></div>
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
          <span
            v-for="rose in homeRoses"
            :key="rose.id"
            class="floating-rose"
            :style="{
              left: `${rose.left}%`,
              top: `${rose.top}%`,
              animationDelay: `${rose.delay}s`,
              fontSize: `${rose.size}px`
            }"
          >🌹</span>
        </div>
        <div class="hero-text">
          <div class="hero-badge">A poetic portrait</div>
          <p class="eyebrow">A luminous portrait</p>
          <h1>Gilanah</h1>
          <p class="lead">
            Gilanah is a soft-hearted dreamer with a glow that feels part moonlight, part midnight
            city light. She carries warmth, wit, and a quiet magic that makes every conversation
            feel intimate and timeless.
          </p>
          <div class="hero-quote">
            “When tenderness meets light, every moment becomes a memory worth keeping.”
          </div>
          <div class="pill-row">
            <span v-for="item in highlights" :key="item">{{ item }}</span>
          </div>
        </div>

        <div class="hero-illustration">
          <div class="hero-actions-side">
            <button class="secondary-btn small-btn" @click="toggleMute">
              <span>{{ isMuted ? 'Unmute song' : 'Mute song' }}</span>
            </button>
            <button class="primary-btn" @click="openAbout">
              <span>Read her story</span>
              <small>Tap to enter her world</small>
            </button>
            <button class="secondary-btn" @click="openImages">
              <span>View her images</span>
              <small>Browse the full gallery</small>
            </button>
            <button class="secondary-btn" @click="openFinal">
              <span>Final page</span>
              <small>End with a romantic note</small>
            </button>
          </div>
          <div class="hero-image-panel">
            <img src="/gigi1.png" alt="Gilanah portrait" />
            <div class="hero-image-overlay">
              <p class="mini-label">Heart of the future</p>
              <h2>Where grace meets glow</h2>
            </div>
          </div>
        </div>
        <div class="section-divider">
          <span class="section-divider-icon">✦</span>
        </div>

        <section class="details-grid">
          <article class="panel">
            <h2>About Gilanah</h2>
            <p>
              She is the kind of person who listens like a soft signal in the dark, notices small
              wonders, and leaves behind a calm, sweet feeling long after she has gone.
            </p>
          </article>

          <article class="panel">
            <h2>Why she feels unforgettable</h2>
            <ul>
              <li v-for="trait in traits" :key="trait">{{ trait }}</li>
            </ul>
          </article>

          <article class="panel">
            <h2>Her favorite little joys</h2>
            <ul>
              <li v-for="thing in favoriteThings" :key="thing">{{ thing }}</li>
            </ul>
          </article>
        </section>

        <div class="section-divider">
          <span class="section-divider-icon">✦</span>
        </div>

        <section class="gallery-section">
          <div class="gallery-heading">
            <p class="eyebrow">A collection of her glow</p>
            <h2>Moments that feel like poetry</h2>
          </div>

          <div class="gallery-grid">
            <article v-for="item in galleryItems" :key="item.title" class="photo-card">
              <img class="photo-bg" :src="item.image" :alt="item.title" loading="lazy" />
              <div class="photo-overlay">
                <h3>{{ item.title }}</h3>
                <p>{{ item.text }}</p>
              </div>
            </article>
          </div>
        </section>

        <section class="closing">
          <p>
            In a world that moves fast, Gilanah feels like a warm light in the future — romantic,
            tender, and beautifully alive.
          </p>
        </section>
      </section>
    </div>

    <div v-else-if="currentView === 'about'" key="about" class="page-transition-wrapper">
      <AboutPage @close="goHome" :active="currentView === 'about'" />
    </div>
    <div v-else-if="currentView === 'images'" key="images" class="page-transition-wrapper">
      <ImagePage @close="goHome" :active="currentView === 'images'" />
    </div>
    <div v-else key="final" class="page-transition-wrapper">
      <FinalPage @close="goHome" :active="currentView === 'final'" />
    </div>
    </Transition>
  </main>
</template>
