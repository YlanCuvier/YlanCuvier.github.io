<template>
  <span :class="badgeClasses">
    <slot />
  </span>
</template>

<script>
export default {
  name: 'InspiraBadge',
  props: {
    variant: {
      type: String,
      default: 'default',
      validator: (value) => ['default', 'success', 'warning', 'error'].includes(value)
    },
    size: {
      type: String,
      default: 'md',
      validator: (value) => ['sm', 'md', 'lg'].includes(value)
    }
  },
  computed: {
    badgeClasses() {
      const baseClasses = 'inline-flex items-center rounded-full font-medium'
      const variantClasses = {
        default: 'bg-gray-100 text-gray-800',
        success: 'bg-green-100 text-green-800',
        warning: 'bg-yellow-100 text-yellow-800',
        error: 'bg-red-100 text-red-800'
      }
      const sizeClasses = {
        sm: 'px-2 py-1 text-xs',
        md: 'px-2.5 py-0.5 text-sm',
        lg: 'px-3 py-1 text-base'
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
