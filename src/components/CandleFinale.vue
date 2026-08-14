<script setup>
import { computed, onBeforeUnmount, ref } from 'vue'
import { BadgeCheck, Sparkles } from 'lucide-vue-next'

const props = defineProps({ birthday: { type: Object, required: true } })
const emit = defineEmits(['celebrate'])

const progress = ref(0)
const complete = ref(false)
let frame
let startedAt = 0

const buttonLabel = computed(() => complete.value ? '愿望已送达宇宙' : progress.value > 0 ? '别松手，愿望正在上传…' : '长按 2.2 秒，替她吹灭蜡烛')

function beginHold(event) {
  if (complete.value) return
  event.currentTarget.setPointerCapture?.(event.pointerId)
  startedAt = performance.now() - progress.value * 2200
  const tick = (now) => {
    progress.value = Math.min((now - startedAt) / 2200, 1)
    if (progress.value >= 1) {
      complete.value = true
      emit('celebrate')
      return
    }
    frame = requestAnimationFrame(tick)
  }
  frame = requestAnimationFrame(tick)
}

function cancelHold() {
  if (complete.value) return
  cancelAnimationFrame(frame)
  progress.value = 0
}

onBeforeUnmount(() => cancelAnimationFrame(frame))
</script>

<template>
  <section class="finale-section section-pad" :class="{ celebrated: complete }">
    <div class="finale-grid" aria-hidden="true"></div>
    <div class="section-label light"><span>06</span> FINAL EXPERIMENT · MAKE A WISH</div>

    <div class="finale-layout">
      <div class="finale-ritual">
        <p class="eyebrow">THE MOST IMPORTANT PROTOCOL</p>
        <h2>最后一项实验：<br>把愿望交给宇宙。</h2>
        <p>请保持一点点认真，再保持很多很多期待。</p>

        <div class="birthday-dish" aria-label="带有 22 数字蜡烛的培养皿蛋糕">
          <span class="dish-rim"></span>
          <span class="cake-base"></span>
          <span class="candle candle-two candle-left"><i></i><b>2</b></span>
          <span class="candle candle-two candle-right"><i></i><b>2</b></span>
          <span v-for="n in 11" :key="n" class="culture-speck" :style="{ '--n': n }"></span>
          <span class="cake-label">CULTURE NO. 220814</span>
        </div>

        <button
          class="hold-button"
          :class="{ done: complete }"
          type="button"
          @pointerdown="beginHold"
          @pointerup="cancelHold"
          @pointercancel="cancelHold"
          @lostpointercapture="cancelHold"
        >
          <span class="hold-fill" :style="{ transform: `scaleX(${progress})` }"></span>
          <Sparkles :size="19" />
          <span>{{ buttonLabel }}</span>
        </button>
      </div>

      <div class="finale-portrait">
        <img :src="birthday.finalePhoto.src" :alt="birthday.finalePhoto.alt" :style="{ objectPosition: birthday.finalePhoto.position }" loading="lazy" />
        <div class="portrait-label"><span>FINAL PLATE</span>{{ birthday.finalePhoto.caption }}</div>
      </div>
    </div>

    <Transition name="wish-reveal">
      <div v-if="complete" class="final-reveal">
        <div class="final-message">
          <span>EXPERIMENT CONCLUDED · RESULT SIGNIFICANT</span>
          <h2>22 岁的你，<br>值得所有显著的好运。</h2>
          <p v-for="line in birthday.finalWish" :key="line">{{ line }}</p>
          <strong>生日快乐，{{ birthday.name }}。</strong>
          <small>愿望已提交宇宙，正在加急审核。</small>
        </div>

        <aside class="agent-license" aria-labelledby="agent-license-title">
          <div class="agent-license-header">
            <div>
              <span>ADDENDUM 22-A · AGENT ACCESS PROTOCOL</span>
              <h3 id="agent-license-title">{{ birthday.agentLicense.title }}</h3>
              <small>CERTIFICATE NO. {{ birthday.agentLicense.number }}</small>
            </div>
            <BadgeCheck :size="38" aria-hidden="true" />
          </div>

          <dl class="agent-license-fields">
            <div v-for="field in birthday.agentLicense.fields" :key="field.label">
              <dt>{{ field.label }}</dt>
              <dd>{{ field.value }}</dd>
            </div>
          </dl>

          <p>{{ birthday.agentLicense.statement }}</p>

          <div class="agent-license-status">
            <span><i aria-hidden="true"></i>{{ birthday.agentLicense.status }}</span>
            <span>{{ birthday.agentLicense.revocation }}</span>
          </div>
        </aside>
      </div>
    </Transition>
  </section>
</template>
