<script setup>
import { onBeforeUnmount, ref, watch } from 'vue'
import { Dna, Sparkles } from 'lucide-vue-next'

const props = defineProps({
  phase: { type: String, required: true },
  birthday: { type: Object, required: true },
})

const emit = defineEmits(['start', 'complete'])
const visibleLines = ref([])
let timer

watch(() => props.phase, (phase) => {
  if (phase !== 'scanning') return
  let index = 0
  visibleLines.value = [props.birthday.scanLines[0]]
  timer = window.setInterval(() => {
    index += 1
    if (index < props.birthday.scanLines.length) {
      visibleLines.value.push(props.birthday.scanLines[index])
      return
    }
    window.clearInterval(timer)
    window.setTimeout(() => emit('complete'), 520)
  }, 360)
})

onBeforeUnmount(() => window.clearInterval(timer))
</script>

<template>
  <section class="launch-gate" :class="{ 'is-scanning': phase === 'scanning' }">
    <div class="gate-grid" aria-hidden="true"></div>
    <div class="gate-strand gate-strand-left" aria-hidden="true">
      <i v-for="n in 9" :key="n"></i>
    </div>
    <div class="gate-panel">
      <div class="system-label"><Dna :size="15" /> {{ birthday.edition }}</div>

      <template v-if="phase === 'gate'">
        <p class="gate-eyebrow">AUG 14 · CLASSIFIED CELEBRATION</p>
        <h1>检测到今日限定<br><em>稀有生命体</em></h1>
        <p class="gate-intro">一项关于她为什么值得被隆重庆祝的非严谨研究，即将公布。</p>
        <button class="launch-button" type="button" @click="emit('start')">
          <Sparkles :size="20" />
          <span>批准她升级到 {{ birthday.age }} 岁</span>
        </button>
        <p class="gate-footnote">本次升级不可撤销，但会变得更酷。</p>
      </template>

      <div v-else class="scan-console" aria-live="polite">
        <div class="scan-orbit" aria-hidden="true">
          <span>{{ birthday.age }}</span>
        </div>
        <div class="scan-copy">
          <p v-for="(line, index) in visibleLines" :key="line" :class="{ success: index === visibleLines.length - 1 }">
            <span>{{ String(index + 1).padStart(2, '0') }}</span>{{ line }}
          </p>
          <i class="scan-cursor"></i>
        </div>
      </div>
    </div>
  </section>
</template>
