<template>
  <div :class="alertClasses" role="alert">
    <div class="flex">
      <div class="flex-shrink-0">
        <component :is="iconComponent" class="h-5 w-5" />
      </div>
      <div class="ml-3">
        <p class="text-sm font-medium">{{ message }}</p>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'InspiraAlert',
  props: {
    type: {
      type: String,
      default: 'info',
      validator: (value) => ['success', 'warning', 'error', 'info'].includes(value)
    },
    message: {
      type: String,
      required: true
    }
  },
  computed: {
    alertClasses() {
      const baseClasses = 'rounded-lg p-4'
      const typeClasses = {
        success: 'bg-green-50 border border-green-200 text-green-800',
        warning: 'bg-yellow-50 border border-yellow-200 text-yellow-800',
        error: 'bg-red-50 border border-red-200 text-red-800',
        info: 'bg-blue-50 border border-blue-200 text-blue-800'
      }

      return [baseClasses, typeClasses[this.type]].join(' ')
    },
    iconComponent() {
      const icons = {
        success: 'CheckCircleIcon',
        warning: 'ExclamationTriangleIcon',
        error: 'XCircleIcon',
        info: 'InformationCircleIcon'
      }
      return icons[this.type] || 'InformationCircleIcon'
    }
  }
}
</script>
