<template>
  <div class="inspira-card p-6">
    <div class="mb-4">
      <h3 class="text-lg font-semibold text-white mb-2">{{ title }}</h3>
      <p class="text-gray-400 text-sm">{{ description }}</p>
    </div>
    <slot />
  </div>
</template>

<script>
export default {
  name: 'InspiraCard',
  props: {
    title: {
      type: String,
      required: true
    },
    description: {
      type: String,
      required: true
    }
  }
}
</script>
