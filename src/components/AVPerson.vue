<script setup lang='ts'>
import { computed, ref, useId, watch } from 'vue'
import { vMaska } from 'maska/vue'

type MaskaDetail = {
  masked: string
  unmasked: string
  completed: boolean
}

const props = defineProps<{
  avatarSrc: string
  label?: string
  caption?: string
  altText?: string
}>()

const model = defineModel<number | null>({ required: true })

const inputId = useId()
const captionId = useId()

const inputValue = ref('')
const isFocused = ref(false)
const isHovered = ref(false)

const sharedInputLayerClass = "col-start-1 row-start-1 box-border h-11 min-w-[72px] rounded-[6px] py-2 pr-4 pl-2 text-[18px] font-medium leading-[22px] tracking-[0] text-[#1E0E4C] [font-family:Inter,sans-serif]"
const mirrorClass = `${sharedInputLayerClass} invisible pointer-events-none whitespace-pre border-transparent`
const inputClass = `${sharedInputLayerClass} outline-none placeholder:font-medium placeholder:leading-[22px] placeholder:tracking-[0] caret-[#3D06D7]`

const componentState = computed<'default' | 'active'>(() => {
  return isFocused.value ? 'active' : 'default'
})

const visualState = computed<'default' | 'hover' | 'active'>(() => {
  if (isFocused.value) {
    return 'active'
  }

  if (isHovered.value) {
    return 'hover'
  }

  return 'default'
})

const mirrorStateClass = computed(() => {
  return visualState.value === 'active' ? 'border-[1.5px]' : 'border'
})

const inputStateClass = computed(() => {
  if (visualState.value === 'active') {
    return 'border-[1.5px] border-[#906FEE]'
  }

  if (visualState.value === 'hover') {
    return 'border border-[#AA9DCE]'
  }

  return 'border border-[#CFCADF]'
})

const inputTextClass = computed(() => {
  return visualState.value === 'active' ? 'text-[#1E0E4C]' : 'text-[rgb(30_14_76_/_0.3)]'
})

const inputPlaceholderClass = computed(() => {
  return visualState.value === 'active'
    ? 'placeholder:text-[#1E0E4C]'
    : 'placeholder:text-[rgb(30_14_76_/_0.3)]'
})

const labelClass = computed(() => {
  return visualState.value === 'active' ? 'text-[#3D06D7]' : 'text-[#1E0E4C]'
})

const captionClass = computed(() => {
  return 'text-[#1E0E4C]'
})

const captionText = computed(() => props.caption ?? 'hours old')

const mirrorValue = computed(() => inputValue.value || '0')

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
  model,
  (nextValue) => {
    const normalizedValue = nextValue === null ? '' : String(nextValue)
    const maskedValue = formatDigits(normalizedValue)

    if (maskedValue !== inputValue.value) {
      inputValue.value = maskedValue
    }
  },
  { immediate: true },
)

function handleMaska(event: CustomEvent<MaskaDetail>) {
  inputValue.value = event.detail.masked
  model.value = event.detail.unmasked === '' ? null : Number(event.detail.unmasked)
}
</script>

<template>
  <div class='flex items-center gap-4'>
    <div class='relative h-20 w-20 flex-none'>
      <div
        v-if='componentState === "active"'
        class='pointer-events-none absolute left-1/2 top-1/2 h-[88px] w-[88px] -translate-x-1/2 -translate-y-1/2 rounded-full border border-[#3D06D7]'
      />

      <img
        :src='avatarSrc'
        :alt='props.altText ?? ""'
        class='h-20 w-20 rounded-full object-cover'
      >
    </div>

    <div class='flex min-w-[162px] flex-col items-start gap-3'>
      <label
        v-if='props.label'
        :for='inputId'
        class='font-normal text-[16px] leading-[15px] tracking-[0.02em] [font-family:Koulen,cursive]'
        :class='labelClass'
      >
        {{ props.label }}
      </label>

      <div class='flex h-11 items-center gap-3'>
        <div
          class='inline-grid align-middle'
          @mouseenter='isHovered = true'
          @mouseleave='isHovered = false'
        >
          <span
            aria-hidden='true'
            :class='[mirrorClass, mirrorStateClass]'
          >
            {{ mirrorValue }}
          </span>

          <input
            :id='inputId'
            :aria-describedby='captionId'
            type='text'
            size='1'
            inputmode='numeric'
            autocomplete='off'
            v-maska='maskOptions'
            v-model='inputValue'
            :class='[
              inputClass,
              inputStateClass,
              inputTextClass,
              inputPlaceholderClass,
            ]'
            placeholder='0'
            @maska='handleMaska($event as CustomEvent<MaskaDetail>)'
            @focus='isFocused = true'
            @blur='isFocused = false'
          >
        </div>

        <span
          :id='captionId'
          class='shrink-0 whitespace-nowrap font-normal text-[18px] leading-[22px] [font-family:Inter,sans-serif]'
          :class='captionClass'
        >
          {{ captionText }}
        </span>
      </div>
    </div>
  </div>
</template>