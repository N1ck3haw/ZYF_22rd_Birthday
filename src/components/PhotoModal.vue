<script setup>
import { onBeforeUnmount, watch } from 'vue'
import { X } from 'lucide-vue-next'

const props = defineProps({ photo: { type: Object, default: null } })
const emit = defineEmits(['close'])

function onKeydown(event) {
  if (event.key === 'Escape') emit('close')
}

watch(() => props.photo, (photo) => {
  document.body.classList.toggle('modal-open', Boolean(photo))
  if (photo) window.addEventListener('keydown', onKeydown)
  else window.removeEventListener('keydown', onKeydown)
})

onBeforeUnmount(() => {
  document.body.classList.remove('modal-open')
  window.removeEventListener('keydown', onKeydown)
})
</script>

<template>
  <Teleport to="body">
    <Transition name="modal">
      <div v-if="photo" class="photo-modal" role="dialog" aria-modal="true" :aria-label="photo.caption" @click.self="emit('close')">
        <button class="modal-close icon-button" type="button" aria-label="关闭照片" @click="emit('close')"><X :size="20" /></button>
        <figure>
          <img :src="photo.src" :alt="photo.alt" :style="{ objectPosition: photo.position }" />
          <figcaption>
            <strong>{{ photo.caption }}</strong>
            <span>{{ photo.note }}</span>
          </figcaption>
        </figure>
      </div>
    </Transition>
  </Teleport>
</template>
