<script setup>
import { computed, onBeforeUnmount, onMounted, ref } from 'vue'
import { Dna, FlaskConical, Heart, Microscope, Sparkles } from 'lucide-vue-next'

const props = defineProps({ birthday: { type: Object, required: true } })
const emit = defineEmits(['open-photo', 'toast'])

const microscopeClicks = ref(0)
const microscopeSolved = computed(() => microscopeClicks.value >= 3)
const secretVisible = ref(false)
let observer
let idleTimer

const cellPattern = (() => {
  const glyph = [
    '1110111',
    '0010001',
    '1110111',
    '1000100',
    '1110111',
  ]
  return glyph.flatMap((row, y) => [...row].flatMap((value, x) => value === '1' ? [{ x: 12 + x * 11, y: 19 + y * 15 }] : []))
})()

const reviews = [
  { code: 'R1', title: '关于快乐', copy: '建议接收。愿新一岁的日常，有值得奔赴的热爱，也有毫无理由的开心。', stamp: 'ACCEPTED' },
  { code: 'R2', title: '关于科研', copy: '愿实验少一点玄学，数据多一点显著；培养基不污染，文献一次找全。', stamp: 'HIGH IMPACT' },
  { code: 'R3', title: '关于自己', copy: '请继续保留敏感、勇敢和好奇。慢一点也没关系，你从来不需要按别人的时间表生长。', stamp: 'ESSENTIAL' },
  { code: 'R4', title: '关于未来', copy: '愿你的每一个选择都更靠近自由，每一场未知都带回新的光。', stamp: 'APPROVED' },
]

const wishes = ['实验一次成功', '顺利毕业', '睡眠充足', '钱包变胖', '永远开心', '被爱包围']

function tapMicroscope() {
  if (microscopeSolved.value) return
  microscopeClicks.value += 1
  if (microscopeSolved.value) emit('toast', '显微镜彩蛋：细胞已自动排列成 22')
  else emit('toast', `显微镜校准中 ${microscopeClicks.value}/3`)
}

function revealSecret() {
  secretVisible.value = !secretVisible.value
}

function resetIdle() {
  window.clearTimeout(idleTimer)
  idleTimer = window.setTimeout(() => emit('toast', '观测对象疑似正在偷偷许愿。'), 22000)
}

onMounted(() => {
  observer = new IntersectionObserver((entries) => {
    entries.forEach((entry) => {
      if (entry.isIntersecting) entry.target.classList.add('is-visible')
    })
  }, { threshold: 0.16 })
  document.querySelectorAll('.reveal').forEach((element) => observer.observe(element))
  ;['pointerdown', 'scroll', 'keydown'].forEach((event) => window.addEventListener(event, resetIdle, { passive: true }))
  resetIdle()
})

onBeforeUnmount(() => {
  observer?.disconnect()
  window.clearTimeout(idleTimer)
  ;['pointerdown', 'scroll', 'keydown'].forEach((event) => window.removeEventListener(event, resetIdle))
})
</script>

<template>
  <div class="story-wrap">
    <section id="profile" class="profile-band section-pad">
      <div class="section-label reveal"><span>01</span> RARE SPECIMEN PROFILE</div>
      <div class="profile-layout">
        <div class="profile-heading reveal">
          <p>今日唯一研究对象</p>
          <h2>{{ birthday.name }}</h2>
          <span>{{ birthday.dedication }}</span>
        </div>
        <dl class="profile-data reveal">
          <div><dt>AGE / 版本</dt><dd>{{ birthday.age }}.0</dd></div>
          <div><dt>FIELD / 栖息领域</dt><dd>{{ birthday.identity }}</dd></div>
          <div><dt>RARITY / 稀有度</dt><dd>宇宙限定</dd></div>
          <div><dt>STATUS / 当前状态</dt><dd>闪闪发光</dd></div>
        </dl>
      </div>
      <div class="profile-verdict reveal">
        <span>EXPERIMENTAL VERDICT</span>
        <p>她不是标准答案。她是那个让所有答案都变得更有趣的变量。</p>
      </div>
    </section>

    <section class="microscope-section section-pad">
      <div class="section-label light reveal"><span>02</span> MICROSCOPIC EVIDENCE</div>
      <div class="microscope-layout">
        <div class="microscope-copy reveal">
          <p class="eyebrow">A TOTALLY SERIOUS STUDY</p>
          <h2>把她放大 2200 倍，<br>还是会发现很多可爱。</h2>
          <p>研究团队试图解释她的快乐、勇敢和好奇心从何而来。遗憾的是，现有仪器只能得出一个结论：天生的。</p>
          <button class="text-button" type="button" @click="tapMicroscope">
            <Microscope :size="19" /> 点击显微镜校准 {{ Math.min(microscopeClicks, 3) }}/3
          </button>
        </div>

        <button class="microscope-view reveal" type="button" aria-label="点击显微镜三次查看隐藏图案" @click="tapMicroscope">
          <span class="scope-ring"></span>
          <span
            v-for="(cell, index) in cellPattern"
            :key="index"
            class="cell-dot"
            :class="{ arranged: microscopeSolved }"
            :style="{
              '--x': `${cell.x}%`,
              '--y': `${cell.y}%`,
              '--rx': `${8 + ((index * 37) % 82)}%`,
              '--ry': `${8 + ((index * 53) % 82)}%`,
              '--delay': `${(index % 7) * 35}ms`,
            }"
          ></span>
          <span class="scope-scale">×2200</span>
          <span v-if="microscopeSolved" class="scope-found">BIRTHDAY CELLS FOUND</span>
        </button>
      </div>

      <div class="stat-strip reveal">
        <div><strong>100%</strong><span>好奇心活性</span></div>
        <div><strong>∞</strong><span>潜在可能性</span></div>
        <button type="button" @click="revealSecret"><strong>p &lt; 0.0001</strong><span>点击查阅显著结论</span></button>
        <div><strong>22×</strong><span>今日好运增幅</span></div>
      </div>
      <Transition name="secret">
        <div v-if="secretVisible" class="secret-result"><Sparkles :size="18" /> 统计学证明：你今天比昨天更值得被爱。</div>
      </Transition>
    </section>

    <section class="memory-section section-pad">
      <div class="section-label reveal"><span>03</span> HIGHLIGHT SLIDES</div>
      <div class="memory-intro reveal">
        <h2>一些值得永久保存的<br>高光切片</h2>
        <p>照片只是时刻的切片，好在笑意、勇气和被爱过的证据，会一直留在里面。</p>
      </div>
      <div class="memory-grid">
        <button
          v-for="(photo, index) in birthday.photos"
          :key="photo.src"
          class="memory-tile reveal"
          :class="`memory-tile-${index + 1}`"
          type="button"
          @click="emit('open-photo', photo)"
        >
          <img :src="photo.src" :alt="photo.alt" :style="{ objectPosition: photo.position, objectFit: photo.fit || 'cover' }" loading="lazy" />
          <span><i>SLIDE {{ String(index + 1).padStart(2, '0') }}</i>{{ photo.caption }}</span>
        </button>
      </div>
      <p class="photo-hint reveal">十段共同记忆，十次快乐复现；点击照片可以查看完整画面。</p>
    </section>

    <section class="review-section section-pad">
      <div class="section-label light reveal"><span>04</span> PEER REVIEW OF HER NEXT YEAR</div>
      <div class="review-heading reveal">
        <Dna :size="34" />
        <h2>来自生活委员会的<br>匿名同行评审</h2>
        <p>审稿意见共四条，均建议无条件接收。</p>
      </div>
      <div class="review-list">
        <article v-for="review in reviews" :key="review.code" class="review-card reveal">
          <span class="review-code">{{ review.code }}</span>
          <div><h3>{{ review.title }}</h3><p>{{ review.copy }}</p></div>
          <strong>{{ review.stamp }}</strong>
        </article>
      </div>
    </section>

    <section class="wish-culture section-pad">
      <div class="section-label reveal"><span>05</span> FUTURE CULTURE PLAN</div>
      <div class="wish-layout">
        <div class="wish-copy reveal">
          <FlaskConical :size="28" />
          <h2>把这些祝愿，<br>放进未来慢慢培养。</h2>
          <p>在光照充足、朋友很多、偶尔偷懒的条件下，预计它们都会长得很好。</p>
        </div>
        <div class="petri-wishes reveal" aria-label="生日祝愿培养皿">
          <button
            v-for="(wish, index) in wishes"
            :key="wish"
            type="button"
            :style="{ '--i': index }"
            @click="emit('toast', `${wish}：已加入重点培养计划`)"
          >{{ wish }}</button>
          <span class="dish-label"><Heart :size="14" fill="currentColor" /> CULTURE: HER BEST YEAR</span>
        </div>
      </div>
    </section>
  </div>
</template>
