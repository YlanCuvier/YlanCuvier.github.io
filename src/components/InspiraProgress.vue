<template>
  <div class="w-full">
    <div class="flex justify-between text-sm text-gray-600 mb-2">
      <span>{{ label }}</span>
      <span>{{ value }}%</span>
    </div>
    <div class="w-full bg-gray-200 rounded-full h-2">
      <div
        :class="progressBarClasses"
        :style="{ width: `${value}%` }"
      ></div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'InspiraProgress',
  props: {
    value: {
      type: Number,
      default: 0,
      validator: (value) => value >= 0 && value <= 100
    },
    label: {
      type: String,
      default: 'Progress'
    },
    variant: {
      type: String,
      default: 'primary',
      validator: (value) => ['primary', 'success', 'warning', 'error'].includes(value)
    }
  },
  computed: {
    progressBarClasses() {
      const baseClasses = 'h-2 rounded-full transition-all duration-300 ease-in-out'
      const variantClasses = {
        primary: 'bg-primary-600',
        success: 'bg-green-500',
        warning: 'bg-yellow-500',
        error: 'bg-red-500'
      }

      return [baseClasses, variantClasses[this.variant]].join(' ')
    }
  }
}
</script>
