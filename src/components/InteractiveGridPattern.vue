<template>
  <div ref="root" class="interactive-grid-root">
    <AnimatedBackground
      class="grid-base"
      :hue="hue"
      :saturation="saturation"
      :brightness="brightness"
      :speed="speed"
    />
    <div class="grid-lines grid-lines-major"></div>
    <div class="grid-lines grid-lines-minor"></div>
    <div class="grid-cursor-glow"></div>
    <div class="grid-vignette"></div>
  </div>
</template>

<script>
import AnimatedBackground from './AnimatedBackground.vue'

export default {
  name: 'InteractiveGridPattern',
  components: {
    AnimatedBackground
  },
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
  data() {
    return {
      pointerCurrent: { x: 0.5, y: 0.5 },
      pointerTarget: { x: 0.5, y: 0.5 },
      pointerFrame: null
    }
  },
  mounted() {
    this.applyGridVars()
    this.applyPointerVars()
    this.startPointerLoop()
    window.addEventListener('pointermove', this.handlePointerMove, { passive: true })
    window.addEventListener('pointerleave', this.handlePointerLeave)
    window.addEventListener('blur', this.handlePointerLeave)
  },
  beforeUnmount() {
    window.removeEventListener('pointermove', this.handlePointerMove)
    window.removeEventListener('pointerleave', this.handlePointerLeave)
    window.removeEventListener('blur', this.handlePointerLeave)
    if (this.pointerFrame) {
      cancelAnimationFrame(this.pointerFrame)
      this.pointerFrame = null
    }
  },
  watch: {
    hue() {
      this.applyGridVars()
    },
    saturation() {
      this.applyGridVars()
    },
    brightness() {
      this.applyGridVars()
    },
    speed() {
      this.applyGridVars()
    }
  },
  methods: {
    clamp(value, min, max) {
      return Math.max(min, Math.min(max, value))
    },
    handlePointerMove(event) {
      const width = window.innerWidth || 1
      const height = window.innerHeight || 1
      this.pointerTarget.x = this.clamp(event.clientX / width, 0, 1)
      this.pointerTarget.y = this.clamp(event.clientY / height, 0, 1)
    },
    handlePointerLeave() {
      this.pointerTarget.x = 0.5
      this.pointerTarget.y = 0.5
    },
    startPointerLoop() {
      const tick = () => {
        const ease = 0.11
        this.pointerCurrent.x += (this.pointerTarget.x - this.pointerCurrent.x) * ease
        this.pointerCurrent.y += (this.pointerTarget.y - this.pointerCurrent.y) * ease
        this.applyPointerVars()
        this.pointerFrame = requestAnimationFrame(tick)
      }

      tick()
    },
    applyPointerVars() {
      const root = this.$refs.root
      if (!root) return

      const px = this.pointerCurrent.x
      const py = this.pointerCurrent.y
      const shiftX = (px - 0.5) * 44
      const shiftY = (py - 0.5) * 44

      root.style.setProperty('--pointer-x', `${(px * 100).toFixed(2)}%`)
      root.style.setProperty('--pointer-y', `${(py * 100).toFixed(2)}%`)
      root.style.setProperty('--grid-major-x', `${(shiftX * 0.55).toFixed(2)}px`)
      root.style.setProperty('--grid-major-y', `${(shiftY * 0.55).toFixed(2)}px`)
      root.style.setProperty('--grid-minor-x', `${(-shiftX * 0.9).toFixed(2)}px`)
      root.style.setProperty('--grid-minor-y', `${(-shiftY * 0.9).toFixed(2)}px`)
    },
    applyGridVars() {
      const root = this.$refs.root
      if (!root) return

      root.style.setProperty('--grid-hue', this.hue)
      root.style.setProperty('--grid-sat', `${Math.max(0, Math.min(1, this.saturation)) * 100}%`)
      root.style.setProperty('--grid-brightness', this.brightness)
      root.style.setProperty('--grid-speed', this.speed)
    }
  }
}
</script>

<style scoped>
.interactive-grid-root {
  position: absolute;
  inset: 0;
  overflow: hidden;
  isolation: isolate;
  --grid-hue: 300;
  --grid-sat: 50%;
  --grid-brightness: 1;
  --grid-speed: 1;
  --pointer-x: 50%;
  --pointer-y: 50%;
  --grid-major-x: 0px;
  --grid-major-y: 0px;
  --grid-minor-x: 0px;
  --grid-minor-y: 0px;
}

.grid-base {
  opacity: 0.35;
  filter: saturate(0.82) brightness(0.68);
}

.grid-lines {
  position: absolute;
  inset: -20%;
  pointer-events: none;
  mix-blend-mode: normal;
}

.grid-lines-major {
  opacity: calc(0.26 * var(--grid-brightness));
  background-image:
    linear-gradient(
      to right,
      hsl(var(--grid-hue) var(--grid-sat) 80% / 0.56) 1px,
      transparent 1px
    ),
    linear-gradient(
      to bottom,
      hsl(var(--grid-hue) var(--grid-sat) 80% / 0.56) 1px,
      transparent 1px
    );
  background-size: 76px 76px;
  background-position:
    var(--grid-major-x) var(--grid-major-y),
    var(--grid-major-x) var(--grid-major-y);
  animation: grid-shift calc(24s / var(--grid-speed)) linear infinite;
}

.grid-lines-minor {
  opacity: calc(0.15 * var(--grid-brightness));
  background-image:
    linear-gradient(
      to right,
      hsl(calc(var(--grid-hue) + 22) var(--grid-sat) 72% / 0.46) 1px,
      transparent 1px
    ),
    linear-gradient(
      to bottom,
      hsl(calc(var(--grid-hue) + 22) var(--grid-sat) 72% / 0.46) 1px,
      transparent 1px
    );
  background-size: 28px 28px;
  background-position:
    var(--grid-minor-x) var(--grid-minor-y),
    var(--grid-minor-x) var(--grid-minor-y);
  animation: grid-shift-alt calc(32s / var(--grid-speed)) linear infinite reverse;
}

.grid-cursor-glow {
  position: absolute;
  inset: -8%;
  pointer-events: none;
  opacity: calc(0.22 * var(--grid-brightness));
  background:
    radial-gradient(
      340px circle at var(--pointer-x) var(--pointer-y),
      hsl(var(--grid-hue) var(--grid-sat) 70% / 0.2) 0%,
      hsl(calc(var(--grid-hue) + 24) var(--grid-sat) 68% / 0.12) 28%,
      transparent 62%
    );
}

.grid-vignette {
  position: absolute;
  inset: 0;
  pointer-events: none;
  background:
    radial-gradient(circle at center, transparent 28%, rgb(2 5 10 / 0.56) 82%),
    radial-gradient(circle at var(--pointer-x) var(--pointer-y), rgb(160 220 255 / 0.1), transparent 36%),
    radial-gradient(circle at 12% 8%, hsl(var(--grid-hue) var(--grid-sat) 72% / 0.14), transparent 48%),
    radial-gradient(circle at 90% 88%, hsl(calc(var(--grid-hue) + 38) var(--grid-sat) 68% / 0.1), transparent 56%);
}

@keyframes grid-shift {
  from {
    transform: translate3d(0, 0, 0);
  }
  to {
    transform: translate3d(76px, 76px, 0);
  }
}

@keyframes grid-shift-alt {
  from {
    transform: translate3d(0, 0, 0);
  }
  to {
    transform: translate3d(28px, 28px, 0);
  }
}
</style>
