<template>
  <div class="animated-background">
    <div class="background-layer layer-1"></div>
    <div class="background-layer layer-2"></div>
    <div class="background-layer layer-3"></div>
    <div class="background-layer layer-4"></div>
  </div>
</template>

<script>
export default {
  name: 'AnimatedBackground',
  props: {
    hue: {
      type: Number,
      default: 300
    },
    saturation: {
      type: Number,
      default: 0.5
    },
    brightness: {
      type: Number,
      default: 1
    },
    speed: {
      type: Number,
      default: 1
    }
  },
  mounted() {
    this.updateColors()
  },
  watch: {
    hue() { this.updateColors() },
    saturation() { this.updateColors() },
    brightness() { this.updateColors() }
  },
  methods: {
    updateColors() {
      const root = this.$el
      root.style.setProperty('--hue', this.hue)
      root.style.setProperty('--saturation', this.saturation)
      root.style.setProperty('--brightness', this.brightness)
      root.style.setProperty('--speed', this.speed)
    }
  }
}
</script>

<style scoped>
.animated-background {
  position: absolute;
  inset: 0;
  overflow: hidden;
  background: hsl(var(--hue, 300), var(--saturation, 50)%, 5%);
}

.background-layer {
  position: absolute;
  inset: 0;
  opacity: 0.1;
}

.layer-1 {
  background:
    radial-gradient(circle at 20% 80%, hsl(var(--hue, 300), var(--saturation, 50)%, 20%) 0%, transparent 50%),
    radial-gradient(circle at 80% 20%, hsl(calc(var(--hue, 300) + 60), var(--saturation, 50)%, 20%) 0%, transparent 50%);
  animation: float calc(20s / var(--speed, 1)) ease-in-out infinite;
}

.layer-2 {
  background:
    radial-gradient(circle at 40% 40%, hsl(calc(var(--hue, 300) + 120), var(--saturation, 50)%, 15%) 0%, transparent 50%),
    radial-gradient(circle at 60% 60%, hsl(calc(var(--hue, 300) + 180), var(--saturation, 50)%, 15%) 0%, transparent 50%);
  animation: float calc(25s / var(--speed, 1)) ease-in-out infinite reverse;
}

.layer-3 {
  background:
    conic-gradient(from 0deg at 50% 50%,
      hsl(var(--hue, 300), var(--saturation, 50)%, 10%) 0deg,
      transparent 60deg,
      hsl(calc(var(--hue, 300) + 90), var(--saturation, 50)%, 10%) 120deg,
      transparent 180deg,
      hsl(calc(var(--hue, 300) + 180), var(--saturation, 50)%, 10%) 240deg,
      transparent 300deg,
      hsl(var(--hue, 300), var(--saturation, 50)%, 10%) 360deg
    );
  animation: rotate calc(30s / var(--speed, 1)) linear infinite;
}

.layer-4 {
  background:
    linear-gradient(45deg,
      hsl(var(--hue, 300), var(--saturation, 50)%, 8%) 0%,
      transparent 25%,
      hsl(calc(var(--hue, 300) + 60), var(--saturation, 50)%, 8%) 50%,
      transparent 75%,
      hsl(var(--hue, 300), var(--saturation, 50)%, 8%) 100%
    );
  animation: wave calc(15s / var(--speed, 1)) ease-in-out infinite;
}

@keyframes float {
  0%, 100% {
    transform: translate(0, 0) scale(1);
  }
  33% {
    transform: translate(30px, -30px) scale(1.1);
  }
  66% {
    transform: translate(-20px, 20px) scale(0.9);
  }
}

@keyframes rotate {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}

@keyframes wave {
  0%, 100% {
    transform: translateX(0) skewX(0deg);
  }
  50% {
    transform: translateX(20px) skewX(5deg);
  }
}
</style>
