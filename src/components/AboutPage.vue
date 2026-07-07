<script setup>
import { inject, onMounted, ref } from 'vue'

const props = defineProps({
  active: Boolean
})

const emit = defineEmits(['close'])
const petals = ref([])
const sparkles = ref([])

const { isMuted, toggleMute } = inject('audioManager')

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

onMounted(() => {
  burstPetals()
})
</script>

<template>
  <section class="about-page" @click="burstPetals">
    <div class="rose-layer" aria-hidden="true">
      <span class="floating-rose" style="left: 8%; top: 12%; font-size: 24px; animation-delay: 0.2s">🌹</span>
      <span class="floating-rose" style="left: 82%; top: 16%; font-size: 20px; animation-delay: 1.4s">🌹</span>
      <span class="floating-rose" style="left: 72%; top: 78%; font-size: 19px; animation-delay: 2.2s">🌹</span>
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

    <div class="about-card">
      <div class="card-top">
        <button class="back-btn" @click.stop="emit('close')">Back</button>
        <button class="primary-btn soundtrack-btn" @click="toggleMute">
          <span>{{ isMuted ? 'Unmute' : 'Mute' }}</span>
        </button>
      </div>
      <p class="eyebrow">About Gilanah</p>
      <h2>A heart with quiet starlight</h2>
      <p>
        Gilanah is the kind of person who makes the world feel softer. She is compassionate,
        dependable, and gentle in a way that helps people feel understood.
      </p>
      <ul class="traits-list">
        <li>Compassionate</li>
        <li>Dependable</li>
        <li>Creative</li>
        <li>Patient</li>
        <li>Optimistic</li>
        <li>Warm-hearted</li>
        <li>Thoughtful</li>
        <li>Loyal</li>
      </ul>
      <p class="about-note">
        She carries romance in ordinary moments — a warm smile, a sincere laugh, and the feeling
        that someone truly sees you.
      </p>
    </div>
  </section>
</template>
