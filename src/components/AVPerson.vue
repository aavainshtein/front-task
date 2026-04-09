<script setup lang='ts'>
import { computed, onBeforeUnmount, ref, watch } from 'vue'
import { vMaska } from 'maska/vue'

type MaskaDetail = {
  masked: string
  unmasked: string
  completed: boolean
}

const props = defineProps<{
  avatarSrc: string
  hours: number | null
  debounce?: number
  inputWidthClass?: string
}>()

const emits = defineEmits<{
  (e: 'change', value: number | null): void
}>()

const inputValue = ref('')
const isFocused = ref(false)

const sharedInputLayerClass = "col-start-1 row-start-1 box-border h-11 min-w-[72px] rounded-[6px] py-2 pr-4 pl-2 text-[18px] font-medium leading-[22px] tracking-[0] text-[#1E0E4C] [font-family:Inter,sans-serif]"
const mirrorClass = `${sharedInputLayerClass} invisible pointer-events-none whitespace-pre border-transparent`
const inputClass = `${sharedInputLayerClass} outline-none transition-[border-color,border-width,opacity] placeholder:text-[#1E0E4C] caret-[#3D06D7]`

const inputStateClass = computed(() => {
  return isFocused.value ? 'border-[1.5px] border-[#906FEE] opacity-100' : 'border border-[#CFCADF] opacity-30'
})

const mirrorStateClass = computed(() => {
  return isFocused.value ? 'border-[1.5px]' : 'border'
})

const maskOptions = {
  mask: '9 99#',
  reversed: true,
  tokens: {
    9: {
      pattern: /[0-9]/,
      repeated: true,
    },
  },
}

function formatDigits(value: string) {
  return value.replace(/\B(?=(\d{3})+(?!\d))/g, ' ')
}

watch(
  () => props.hours,
  (nextValue) => {
    const normalizedValue = nextValue === null ? '' : String(nextValue)
    const maskedValue = formatDigits(normalizedValue)

    if (maskedValue !== inputValue.value) {
      inputValue.value = maskedValue
    }
  },
  { immediate: true },
)


let timer: ReturnType<typeof setTimeout> | undefined

function emitChange(value: number | null) {
  const debounceMs = props.debounce ?? 0

  if (timer) {
    clearTimeout(timer)
  }

  if (debounceMs <= 0) {
    emits('change', value)
    return
  }

  timer = setTimeout(() => {
    emits('change', value)
  }, debounceMs)
}

function handleMaska(event: CustomEvent<MaskaDetail>) {
  inputValue.value = event.detail.masked
  emitChange(event.detail.unmasked === '' ? null : Number(event.detail.unmasked))
}

onBeforeUnmount(() => {
  if (timer) {
    clearTimeout(timer)
  }
})
</script>

<template>
  <div class='flex flex-shrink-0 min-w-[32px] items-center gap-3'>
    <img
      :src='avatarSrc'
      alt=''
      class='h-14 w-14 rounded-full border-2 object-cover transition-colors'
      :class='isFocused ? "border-violet-500" : "border-gray-300"'
    >

    <div>
      <div class='inline-grid align-middle'>
        <span
          aria-hidden='true'
          :class='[mirrorClass, mirrorStateClass]'
        >
          {{ inputValue || '0' }}
        </span>

        <input
          type='text'
          size='1'
          inputmode='numeric'
          autocomplete='off'
          v-maska='maskOptions'
          v-model='inputValue'
          :class='[
            inputClass,
            inputStateClass,
          ]'
          placeholder='0'
          @maska='handleMaska($event as CustomEvent<MaskaDetail>)'
          @focus='isFocused = true'
          @blur='isFocused = false'
        >
      </div>
    </div>
  </div>
</template>