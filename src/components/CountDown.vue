<script setup lang="ts">
import { computed } from 'vue';

interface Props {
  timer: number
  max: number
}

const props = defineProps<Props>()

const bar = computed(() => {
  const barNumber = (props.timer - props.max) * (100 / props.max)

  return barNumber < 0 ? barNumber : 0
})

const textColor = computed(() => {
  return bar.value > - 66 ? 'text-emerald-600' : 'text-red-500'
})

const displayTimer = computed(() => {
  return props.timer > 0 ? props.timer : 0
})
</script>

<template>
  <div class="relative size-32">
    <svg class="size-full -rotate-90" viewBox="0 0 36 36" xmlns="http://www.w3.org/2000/svg">
      <circle cx="18" cy="18" r="16" fill="none" class="stroke-current text-gray-700" stroke-width="2"></circle>
      <circle v-show="timer > 0" cx="18" cy="18" r="16" fill="none"
        class="stroke-current transition-all ease-in-out duration-200" :class="textColor" stroke-width="2"
        stroke-dasharray="100" :stroke-dashoffset="bar" stroke-linecap="round">
      </circle>
    </svg>

    <div class="absolute top-1/2 start-1/2 transform -translate-y-1/2 -translate-x-1/2">
      <span class="text-center text-4xl font-bold transition-all duration-200 ease-in-out" :class="textColor">
        {{ displayTimer }}
      </span>
    </div>
  </div>
</template>
