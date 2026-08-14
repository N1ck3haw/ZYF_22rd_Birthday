<script setup>
const props = defineProps({
  collected: { type: Array, required: true },
})

defineEmits(['collect'])

const genes = [
  { id: 'A1', letter: 'A', x: '4%', y: '31%' },
  { id: 'T2', letter: 'T', x: '92%', y: '22%' },
  { id: 'C3', letter: 'C', x: '7%', y: '73%' },
  { id: 'G4', letter: 'G', x: '89%', y: '63%' },
  { id: 'X5', letter: '✦', x: '53%', y: '88%' },
]
</script>

<template>
  <div class="ambient-lab" aria-label="隐藏生日基因彩蛋">
    <div class="dna-rail dna-rail-left"><i v-for="n in 15" :key="n"></i></div>
    <div class="dna-rail dna-rail-right"><i v-for="n in 15" :key="n"></i></div>
    <button
      v-for="gene in genes"
      :key="gene.id"
      class="hidden-gene"
      :class="{ collected: props.collected.includes(gene.id) }"
      :style="{ left: gene.x, top: gene.y }"
      type="button"
      :aria-label="`隐藏的生日基因 ${gene.letter}`"
      @click="$emit('collect', gene.id)"
    >{{ gene.letter }}</button>
  </div>
</template>
