<template>
  <canvas ref="canvas" class="w-full h-full"></canvas>
</template>

<script>
export default {
  name: 'ShaderToy',
  props: {
    shaderCode: {
      type: String,
      required: true
    },
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
      gl: null,
      program: null,
      animationId: null,
      startTime: 0,
      mouse: { x: 0, y: 0, z: 0 }
    }
  },
  mounted() {
    this.initWebGL()
    this.startAnimation()
    this.setupEventListeners()
  },
  beforeUnmount() {
    if (this.animationId) {
      cancelAnimationFrame(this.animationId)
    }
    this.removeEventListeners()
  },
  methods: {
    initWebGL() {
      const canvas = this.$refs.canvas
      this.gl = canvas.getContext('webgl') || canvas.getContext('experimental-webgl')

      if (!this.gl) {
        console.error('WebGL not supported')
        this.createFallbackBackground()
        return
      }

      this.resizeCanvas()
      this.createShaderProgram()
    },

    resizeCanvas() {
      const canvas = this.$refs.canvas
      const rect = canvas.getBoundingClientRect()
      canvas.width = rect.width
      canvas.height = rect.height
      if (this.gl) {
        this.gl.viewport(0, 0, canvas.width, canvas.height)
      }
    },

    createFallbackBackground() {
      const canvas = this.$refs.canvas
      const ctx = canvas.getContext('2d')
      this.resizeCanvas()

      const animate = () => {
        const time = (performance.now() - this.startTime) / 1000

        ctx.fillStyle = '#000000'
        ctx.fillRect(0, 0, canvas.width, canvas.height)

        for (let i = 0; i < 50; i++) {
          const x = (Math.sin(time * 0.5 + i * 0.1) * 0.5 + 0.5) * canvas.width
          const y = (Math.cos(time * 0.3 + i * 0.2) * 0.5 + 0.5) * canvas.height
          const size = Math.sin(time + i) * 10 + 20

          ctx.fillStyle = `hsl(${this.hue}, ${this.saturation * 100}%, ${this.brightness * 30}%)`
          ctx.beginPath()
          ctx.arc(x, y, size, 0, Math.PI * 2)
          ctx.fill()
        }

        this.animationId = requestAnimationFrame(animate)
      }

      animate()
    },

    createShaderProgram() {
      const vertexShaderSource = `
        attribute vec2 a_position;
        void main() {
          gl_Position = vec4(a_position, 0.0, 1.0);
        }
      `

      const fragmentShaderSource = `
        precision mediump float;
        uniform vec2 iResolution;
        uniform float iTime;
        uniform vec3 iMouse;
        uniform float iHue;
        uniform float iSaturation;
        uniform float iBrightness;
        uniform float iSpeed;

        float noise(vec2 p) {
          return smoothstep(-0.5, 0.9, sin((p.x - p.y) * 555.0) * sin(p.y * 1444.0)) - 0.4;
        }

        float fabric(vec2 p) {
          const mat2 m = mat2(1.6, 1.2, -1.2, 1.6);
          float f = 0.4 * noise(p);
          f += 0.3 * noise(p = m * p);
          f += 0.2 * noise(p = m * p);
          return f + 0.1 * noise(m * p);
        }

        float silk(vec2 uv, float t) {
          float s = sin(5.0 * (uv.x + uv.y + cos(2.0 * uv.x + 5.0 * uv.y)) + sin(12.0 * (uv.x + uv.y)) - t);
          s = 0.7 + 0.3 * (s * s * 0.5 + s);
          s *= 0.9 + 0.6 * fabric(uv * min(iResolution.x, iResolution.y) * 0.0006);
          return s * 0.9 + 0.1;
        }

        float silkd(vec2 uv, float t) {
          float xy = uv.x + uv.y;
          float d = (5.0 * (1.0 - 2.0 * sin(2.0 * uv.x + 5.0 * uv.y)) + 12.0 * cos(12.0 * xy)) * cos(5.0 * (cos(2.0 * uv.x + 5.0 * uv.y) + xy) + sin(12.0 * xy) - t);
          return 0.005 * d * (sign(d) + 3.0);
        }

        vec3 hsv2rgb(vec3 c) {
          vec4 K = vec4(1.0, 2.0 / 3.0, 1.0 / 3.0, 3.0);
          vec3 p = abs(fract(c.xxx + K.xyz) * 6.0 - K.www);
          return c.z * mix(K.xxx, clamp(p - K.xxx, 0.0, 1.0), c.y);
        }

        void main() {
          float mr = min(iResolution.x, iResolution.y);
          vec2 uv = gl_FragCoord.xy / mr;

          float t = iTime * iSpeed;
          uv.y += 0.03 * sin(8.0 * uv.x - t);

          if (iMouse.z > 1.0)
            uv += smoothstep(0.5, 0.0, distance(iMouse.xy / mr, uv)) * 0.08;

          float s = sqrt(silk(uv, t));
          float d = silkd(uv, t);

          vec3 c = vec3(s);
          c += 0.7 * vec3(1, 0.83, 0.6) * d;
          c *= 1.0 - max(0.0, 0.8 * d);

          c = pow(c, 0.3 / vec3(0.52, 0.5, 0.4));
          c = 1.0 - c;

          c *= iBrightness;

          vec3 hsv = vec3(iHue / 360.0, iSaturation, 1.0);
          vec3 rgb = hsv2rgb(hsv);
          c = mix(c, c * rgb, iSaturation);

          gl_FragColor = vec4(c, 1.0);
        }
      `

      const vertexShader = this.createShader(this.gl.VERTEX_SHADER, vertexShaderSource)
      const fragmentShader = this.createShader(this.gl.FRAGMENT_SHADER, fragmentShaderSource)

      if (!vertexShader || !fragmentShader) return

      this.program = this.gl.createProgram()
      this.gl.attachShader(this.program, vertexShader)
      this.gl.attachShader(this.program, fragmentShader)
      this.gl.linkProgram(this.program)

      if (!this.gl.getProgramParameter(this.program, this.gl.LINK_STATUS)) {
        console.error('Shader program failed to link:', this.gl.getProgramInfoLog(this.program))
        this.createFallbackBackground()
        return
      }

      this.setupGeometry()
    },

    createShader(type, source) {
      const shader = this.gl.createShader(type)
      this.gl.shaderSource(shader, source)
      this.gl.compileShader(shader)

      if (!this.gl.getShaderParameter(shader, this.gl.COMPILE_STATUS)) {
        console.error('Shader compilation error:', this.gl.getShaderInfoLog(shader))
        this.gl.deleteShader(shader)
        return null
      }

      return shader
    },

    setupGeometry() {
      const positions = new Float32Array([
        -1, -1,
         1, -1,
        -1,  1,
         1,  1
      ])

      const buffer = this.gl.createBuffer()
      this.gl.bindBuffer(this.gl.ARRAY_BUFFER, buffer)
      this.gl.bufferData(this.gl.ARRAY_BUFFER, positions, this.gl.STATIC_DRAW)

      const positionLocation = this.gl.getAttribLocation(this.program, 'a_position')
      this.gl.enableVertexAttribArray(positionLocation)
      this.gl.vertexAttribPointer(positionLocation, 2, this.gl.FLOAT, false, 0, 0)
    },

    setupEventListeners() {
      const canvas = this.$refs.canvas
      canvas.addEventListener('mousemove', this.handleMouseMove)
      canvas.addEventListener('mousedown', this.handleMouseDown)
      canvas.addEventListener('mouseup', this.handleMouseUp)
      window.addEventListener('resize', this.handleResize)
    },

    removeEventListeners() {
      const canvas = this.$refs.canvas
      canvas.removeEventListener('mousemove', this.handleMouseMove)
      canvas.removeEventListener('mousedown', this.handleMouseDown)
      canvas.removeEventListener('mouseup', this.handleMouseUp)
      window.removeEventListener('resize', this.handleResize)
    },

    handleMouseMove(event) {
      const rect = this.$refs.canvas.getBoundingClientRect()
      this.mouse.x = event.clientX - rect.left
      this.mouse.y = event.clientY - rect.top
    },

    handleMouseDown(event) {
      this.mouse.z = 1.0
    },

    handleMouseUp(event) {
      this.mouse.z = 0.0
    },

    handleResize() {
      this.resizeCanvas()
    },

    render() {
      if (!this.gl || !this.program) return

      this.gl.useProgram(this.program)

      const timeLocation = this.gl.getUniformLocation(this.program, 'iTime')
      const resolutionLocation = this.gl.getUniformLocation(this.program, 'iResolution')
      const mouseLocation = this.gl.getUniformLocation(this.program, 'iMouse')
      const hueLocation = this.gl.getUniformLocation(this.program, 'iHue')
      const saturationLocation = this.gl.getUniformLocation(this.program, 'iSaturation')
      const brightnessLocation = this.gl.getUniformLocation(this.program, 'iBrightness')
      const speedLocation = this.gl.getUniformLocation(this.program, 'iSpeed')

      const currentTime = (performance.now() - this.startTime) / 1000

      this.gl.uniform1f(timeLocation, currentTime)
      this.gl.uniform2f(resolutionLocation, this.$refs.canvas.width, this.$refs.canvas.height)
      this.gl.uniform3f(mouseLocation, this.mouse.x, this.mouse.y, this.mouse.z)
      this.gl.uniform1f(hueLocation, this.hue)
      this.gl.uniform1f(saturationLocation, this.saturation)
      this.gl.uniform1f(brightnessLocation, this.brightness)
      this.gl.uniform1f(speedLocation, this.speed)

      this.gl.drawArrays(this.gl.TRIANGLE_STRIP, 0, 4)
    },

    startAnimation() {
      this.startTime = performance.now()
      const animate = () => {
        this.render()
        this.animationId = requestAnimationFrame(animate)
      }
      animate()
    }
  }
}
</script>
