<template>
  <button
    :class="buttonClasses"
    :disabled="disabled"
    @click="$emit('click', $event)"
  >
    <slot />
  </button>
</template>

<script>
export default {
  name: 'InspiraButton',
  props: {
    variant: {
      type: String,
      default: 'primary',
      validator: (value) => ['primary', 'secondary', 'outline'].includes(value)
    },
    size: {
      type: String,
      default: 'md',
      validator: (value) => ['sm', 'md', 'lg'].includes(value)
    },
    disabled: {
      type: Boolean,
      default: false
    }
  },
  emits: ['click'],
  computed: {
    buttonClasses() {
      const baseClasses = 'inspira-button'
      const variantClasses = {
        primary: 'inspira-button-primary',
        secondary: 'inspira-button-secondary',
        outline: 'inspira-button-outline'
      }
      const sizeClasses = {
        sm: 'h-8 px-3 text-xs',
        md: 'h-10 px-4 py-2',
        lg: 'h-12 px-6 text-base'
      }

      return [
        baseClasses,
        variantClasses[this.variant],
        sizeClasses[this.size]
      ].join(' ')
    }
  }
}
</script>
