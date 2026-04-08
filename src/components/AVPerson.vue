<script setup lang='ts'>
import { computed, nextTick, onBeforeUnmount, ref, watch } from 'vue'

const props = defineProps<{
  avatarSrc: string
  value: number | null
  debounce?: number
}>()

const emits = defineEmits<{
  (e: 'change', value: number | null): void
}>()

const rawDigits = ref('')
const isFocused = ref(false)
const mirror = ref<HTMLSpanElement | null>(null)
const inputWidth = ref(72)

const displayValue = computed(() => {
  return rawDigits.value.replace(/\B(?=(\d{3})+(?!\d))/g, ' ')
})

watch(
  () => props.value,
  (nextValue) => {
    const normalizedValue = nextValue === null ? '' : String(nextValue)

    if (normalizedValue !== rawDigits.value) {
      rawDigits.value = normalizedValue
    }
  },
  { immediate: true },
)

watch(
  displayValue,
  () => {
    nextTick(() => {
      if (!mirror.value) return

      inputWidth.value = Math.max(72, mirror.value.offsetWidth + 20)
    })
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

function handleInput(event: Event) {
  const nextDigits = (event.target as HTMLInputElement).value.replace(/\D/g, '')

  rawDigits.value = nextDigits
  emitChange(nextDigits === '' ? null : Number(nextDigits))
}

function handleKeydown(event: KeyboardEvent) {
  const allowedKeys = [
    'Backspace',
    'Delete',
    'ArrowLeft',
    'ArrowRight',
    'ArrowUp',
    'ArrowDown',
    'Tab',
    'Home',
    'End',
  ]

  if (allowedKeys.includes(event.key) || event.ctrlKey || event.metaKey) {
    return
  }

  if (!/^\d$/.test(event.key)) {
    event.preventDefault()
  }
}

onBeforeUnmount(() => {
  if (timer) {
    clearTimeout(timer)
  }
})
</script>

<template>
  <div class='flex items-center gap-3'>
    <img
      :src='avatarSrc'
      alt=''
      class='h-14 w-14 rounded-full border-2 object-cover transition-colors'
      :class='isFocused ? "border-violet-500" : "border-gray-300"'
    >

    <div class='relative inline-block'>
      <span
        ref='mirror'
        aria-hidden='true'
        class='invisible absolute whitespace-pre rounded px-2 py-1 text-lg'
      >{{ displayValue || '0' }}</span>

      <input
        type='text'
        inputmode='numeric'
        autocomplete='off'
        :value='displayValue'
        :style='{ width: `${inputWidth}px` }'
        class='rounded border px-2 py-1 text-lg outline-none transition-colors'
        :class='isFocused ? "border-violet-500" : "border-gray-300"'
        placeholder='0'
        @input='handleInput'
        @keydown='handleKeydown'
        @focus='isFocused = true'
        @blur='isFocused = false'
      >
    </div>
  </div>
</template>