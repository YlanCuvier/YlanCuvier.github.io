<template>
  <div class="relative grid h-[500px] place-content-center">
    <InteractiveGridPattern
      :class="'[mask-image:radial-gradient(350px_circle_at_center,white,transparent)]'"
      :hue="300"
      :saturation="0.8"
      :brightness="0.6"
      :speed="0.5"
    />
  </div>
</template>

<script>
import InteractiveGridPattern from './InteractiveGridPattern.vue'

export default {
  name: 'InteractiveGrid',
  components: {
    InteractiveGridPattern
  }
}
</script>
