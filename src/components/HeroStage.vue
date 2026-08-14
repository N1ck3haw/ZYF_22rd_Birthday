<script setup>
import { onBeforeUnmount, onMounted, ref } from 'vue'
import gsap from 'gsap'
import { ArrowDown, FlaskConical, Sparkles } from 'lucide-vue-next'

const props = defineProps({ birthday: { type: Object, required: true } })
const emit = defineEmits(['open-photo'])
const root = ref(null)
let gsapContext

const cardClasses = ['card-main', 'card-left-top', 'card-right-top', 'card-left-bottom', 'card-right-bottom', 'card-far-right']

onMounted(() => {
  const reduced = window.matchMedia('(prefers-reduced-motion: reduce)').matches
  if (reduced) return
  gsapContext = gsap.context(() => {
    const timeline = gsap.timeline({ defaults: { ease: 'power3.out' } })
    timeline
      .from('.hero-kicker', { y: 18, opacity: 0, duration: 0.55 })
      .from('.age-number', { scale: 0.2, rotation: -12, opacity: 0, duration: 1.15, ease: 'elastic.out(1, .55)' }, '-=.1')
      .from('.hero-title span', { yPercent: 120, opacity: 0, stagger: 0.12, duration: 0.75 }, '-=.65')
      .from('.hero-subtitle, .hero-specimen-tag', { y: 18, opacity: 0, stagger: 0.1, duration: 0.55 }, '-=.35')
      .from('.photo-card', { scale: 0.25, opacity: 0, rotation: () => gsap.utils.random(-25, 25), stagger: 0.14, duration: 0.85, ease: 'back.out(1.9)' }, '-=.15')
      .set('.photo-card', { clearProps: 'transform,opacity' })
      .from('.scroll-invite', { opacity: 0, y: -10, duration: 0.5 }, '-=.1')
  }, root.value)
})

onBeforeUnmount(() => gsapContext?.revert())
</script>

<template>
  <section id="top" ref="root" class="hero-stage">
    <div class="hero-grid" aria-hidden="true"></div>
    <div class="hero-wordmark" aria-label="Birthday Biosystem 22">
      <span class="wordmark-mark">B·22</span>
      <span class="wordmark-copy">RARE SPECIMEN EDITION</span>
    </div>
    <div class="hero-copy">
      <p class="hero-kicker"><FlaskConical :size="15" /> OFFICIAL RELEASE · {{ birthday.birthday }}</p>
      <div class="age-wrap" aria-label="22 岁">
        <span class="age-number">{{ birthday.age }}</span>
        <span class="age-unit">YEARS<br>OF MAGIC</span>
      </div>
      <h1 class="hero-title"><span>HAPPY</span><span>BIRTHDAY</span></h1>
      <p class="hero-subtitle">{{ birthday.heroLine }}</p>
      <div class="hero-specimen-tag"><Sparkles :size="15" /> SPECIMEN: {{ birthday.name }} · STATUS: GLOWING</div>
    </div>

    <div class="photo-storm" aria-label="生日照片预览">
      <button
        v-for="(photo, index) in birthday.photos.slice(0, 6)"
        :key="photo.src"
        class="photo-card"
        :class="cardClasses[index]"
        type="button"
        :aria-label="`放大查看：${photo.caption}`"
        @click="emit('open-photo', photo)"
      >
        <span class="photo-index">0{{ index + 1 }}</span>
        <img :src="photo.src" :alt="photo.alt" :style="{ objectPosition: photo.position, objectFit: photo.fit || 'cover' }" :fetchpriority="index === 0 ? 'high' : 'auto'" />
        <span class="photo-caption">{{ photo.caption }}</span>
      </button>
    </div>

    <a class="scroll-invite" href="#profile">
      <span>开始阅读研究报告</span>
      <ArrowDown :size="18" />
    </a>
  </section>
</template>
