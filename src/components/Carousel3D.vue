<template>
  <div
    class="relative w-full h-full"
    :style="{
      perspective: `${perspective}px`,
    }"
  >
    <div
      ref="wheel"
      class="relative w-full h-full"
      :style="{
        transformStyle: 'preserve-3d',
        transition: 'transform 1s ease',
      }"
      @mousedown="startDrag"
    >
      <div
        v-for="(item, index) in items"
        :key="index"
        class="absolute rounded bg-white overflow-hidden shadow-lg"
        :style="{
          width: `${itemWidth}px`,
          height: `${itemHeight}px`,
          left: '50%',
          top: '50%',
          marginTop: `-${itemHeight / 2}px`,
          transformOrigin: 'left center',
          transform: getItemTransform(index),
          transition: 'transform 0.3s ease',
        }"
      >
        <img
          :src="item.image"
          :alt="item.alt || `Image ${index + 1}`"
          class="w-full h-full object-cover select-none pointer-events-none"
          draggable="false"
        />
      </div>
    </div>

    <div
      class="absolute -bottom-3 left-0 w-full flex justify-center gap-5 z-10"
      v-if="showControls"
    >
      <button
        @click="prevItem"
        class="px-5 py-2 bg-gray-700 text-white rounded hover:bg-gray-600 transition-colors"
      >
        {{ prevText }}
      </button>
      <button
        @click="nextItem"
        class="px-5 py-2 bg-gray-700 text-white rounded hover:bg-gray-600 transition-colors"
      >
        {{ nextText }}
      </button>
    </div>
  </div>
</template>

<script lang="ts">
export default {
  name: 'Carousel3D',
  props: {
    // Items to display in the carousel
    items: {
      type: Array,
      required: true,
      default: () => [],
    },
    // Individual item dimensions
    itemWidth: {
      type: Number,
      default: 180,
    },
    itemHeight: {
      type: Number,
      default: 240,
    },
    // Distance from center to items
    radius: {
      type: Number,
      default: 120,
    },
    // Perspective value for 3D effect
    perspective: {
      type: Number,
      default: 1200,
    },
    // Rotation sensitivity when dragging
    sensitivity: {
      type: Number,
      default: 0.5,
    },
    // Friction factor for inertial movement (0-1)
    friction: {
      type: Number,
      default: 0.95,
    },
    // Scale factor for micro-animation
    pulseScale: {
      type: Number,
      default: 1.02,
    },
    // Duration of micro-animation in ms
    pulseDuration: {
      type: Number,
      default: 300,
    },
    // Show navigation controls
    showControls: {
      type: Boolean,
      default: true,
    },
    // Button text
    prevText: {
      type: String,
      default: 'Previous',
    },
    nextText: {
      type: String,
      default: 'Next',
    },
    // Show info text
    showInfo: {
      type: Boolean,
      default: true,
    },
    // Rotation angle for navigation buttons
    rotationStep: {
      type: Number,
      default: 45,
    },
    // Auto rotate settings
    autoRotate: {
      type: Boolean,
      default: false,
    },
    // Auto rotate speed (degrees per second)
    autoRotateSpeed: {
      type: Number,
      default: 10,
    },
    // Auto rotate direction (true = clockwise, false = counterclockwise)
    autoRotateClockwise: {
      type: Boolean,
      default: false,
    },
  },
  data() {
    return {
      currentAngle: 0,
      isDragging: false,
      startX: 0,
      lastX: 0,
      startAngle: 0,
      velocity: 0,
      lastTimestamp: 0,
      animationId: null,
      autoRotateId: null,
      lastAutoRotateTime: 0,
    }
  },
  mounted() {
    // Prevent browser's default drag behavior
    this.$el.addEventListener('dragstart', (e) => e.preventDefault())

    // Add event listeners for drag
    document.addEventListener('mousemove', this.onDrag)
    document.addEventListener('mouseup', this.stopDrag)
    document.addEventListener('mouseleave', this.stopDrag)

    // Add keyboard controls
    document.addEventListener('keydown', this.handleKeyDown)

    // Start auto-rotate if enabled
    this.startAutoRotateIfEnabled()
  },
  beforeUnmount() {
    // Clean up event listeners
    document.removeEventListener('mousemove', this.onDrag)
    document.removeEventListener('mouseup', this.stopDrag)
    document.removeEventListener('mouseleave', this.stopDrag)
    document.removeEventListener('keydown', this.handleKeyDown)

    // Cancel any ongoing animations
    if (this.animationId) {
      cancelAnimationFrame(this.animationId)
    }

    // Cancel auto-rotate
    this.stopAutoRotate()
  },
  watch: {
    // Watch for changes to autoRotate prop
    autoRotate(newValue) {
      if (newValue) {
        this.startAutoRotate()
      } else {
        this.stopAutoRotate()
      }
    },

    // Watch for changes to autoRotateSpeed prop
    autoRotateSpeed() {
      if (this.autoRotate) {
        // Restart auto-rotate with new speed
        this.stopAutoRotate()
        this.startAutoRotate()
      }
    },
  },
  methods: {
    // Calculate transform for each item, exactly matching the original HTML implementation
    getItemTransform(index) {
      const angle = (index / this.items.length) * 360

      return `
        rotateY(${angle}deg) 
        translateZ(-20px)
        translateX(${this.radius}px)
      `
    },

    // Navigation methods
    prevItem() {
      this.rotateWheel(this.rotationStep)
    },

    nextItem() {
      this.rotateWheel(-this.rotationStep)
    },

    // Simple micro-animation during rotation
    applyMicroAnimation() {
      const cards = this.$el.querySelectorAll('.absolute')

      // Add a small pulse scale animation
      cards.forEach((card) => {
        // Reset any previous inline transform
        const currentTransform = card.style.transform

        // Apply a subtle pulse scale animation
        card.style.transform = `${currentTransform.split('scale')[0]} scale(${this.pulseScale})`

        // Reset after animation completes
        setTimeout(() => {
          card.style.transform = currentTransform
        }, this.pulseDuration)
      })
    },

    // Rotate the wheel
    rotateWheel(degrees) {
      // Stop auto-rotate temporarily
      this.stopAutoRotate()

      // Cancel any ongoing animation
      if (this.animationId) {
        cancelAnimationFrame(this.animationId)
        this.animationId = null
      }

      // Apply subtle micro-animation
      this.applyMicroAnimation()

      // Enable transition for smooth rotation
      if (this.$refs.wheel) {
        this.$refs.wheel.style.transition = 'transform 0.5s ease'
      }

      // Update angle
      this.currentAngle += degrees
      if (this.$refs.wheel) {
        this.$refs.wheel.style.transform = `rotateY(${this.currentAngle}deg)`
      }

      // Reset velocity and animation state
      this.velocity = 0
      this.lastTimestamp = 0

      // Reset transition after animation completes
      setTimeout(() => {
        if (this.$refs.wheel) {
          this.$refs.wheel.style.transition = 'none'
        }

        // Restart auto-rotate if it was enabled
        if (this.autoRotate) {
          this.startAutoRotate()
        }
      }, 500)
    },

    // Mouse drag handlers
    startDrag(e) {
      e.preventDefault()

      // Stop auto-rotate while dragging
      this.stopAutoRotate()

      // Stop any ongoing animation
      if (this.animationId) {
        cancelAnimationFrame(this.animationId)
        this.animationId = null
      }

      this.isDragging = true
      this.startX = e.clientX
      this.lastX = this.startX
      this.startAngle = this.currentAngle
      this.lastTimestamp = 0
      this.velocity = 0

      // Disable transition during drag
      if (this.$refs.wheel) {
        this.$refs.wheel.style.transition = 'none'
      }

      // Change cursor
      document.body.style.cursor = 'grabbing'
    },

    onDrag(e) {
      if (!this.isDragging) return

      // Calculate velocity
      const now = performance.now()
      const elapsed = now - this.lastTimestamp || 16
      const deltaX = e.clientX - this.lastX

      // Update velocity (scaled for rotation)
      this.velocity = (deltaX * 0.3) / elapsed

      // Calculate total rotation
      const totalDeltaX = e.clientX - this.startX
      const newAngle = this.startAngle + totalDeltaX * this.sensitivity

      // Update wheel rotation
      if (this.$refs.wheel) {
        this.$refs.wheel.style.transform = `rotateY(${newAngle}deg)`
      }
      this.currentAngle = newAngle

      // Store position for next calculation
      this.lastX = e.clientX
      this.lastTimestamp = now
    },

    stopDrag() {
      if (!this.isDragging) return

      this.isDragging = false

      // Reset cursor
      document.body.style.cursor = 'default'

      // Start inertia animation
      this.lastTimestamp = 0
      this.animationId = requestAnimationFrame(this.applyInertia)

      // Restart auto-rotate after inertia if it was enabled
      if (this.autoRotate) {
        // Wait for inertia to finish before restarting auto-rotate
        const checkInertiaFinished = () => {
          if (this.animationId) {
            setTimeout(checkInertiaFinished, 100)
          } else {
            this.startAutoRotate()
          }
        }

        setTimeout(checkInertiaFinished, 100)
      }
    },

    // Apply inertial rotation
    applyInertia(timestamp) {
      if (!this.lastTimestamp) {
        this.lastTimestamp = timestamp
        this.animationId = requestAnimationFrame(this.applyInertia)
        return
      }

      const elapsed = timestamp - this.lastTimestamp
      this.lastTimestamp = timestamp

      // Apply friction
      this.velocity *= this.friction

      // Update angle based on velocity
      this.currentAngle += (this.velocity * elapsed) / 16

      if (this.$refs.wheel) {
        this.$refs.wheel.style.transform = `rotateY(${this.currentAngle}deg)`
      }

      // Apply micro-animation at start of inertial movement
      if (Math.abs(this.velocity) > 1.5) {
        this.applyMicroAnimation()
      }

      // Continue animation or stop
      if (Math.abs(this.velocity) > 0.1) {
        this.animationId = requestAnimationFrame(this.applyInertia)
      } else {
        this.lastTimestamp = 0
        this.velocity = 0
        this.animationId = null
      }
    },

    // Keyboard controls
    handleKeyDown(e) {
      if (e.key === 'ArrowLeft') {
        this.prevItem()
      } else if (e.key === 'ArrowRight') {
        this.nextItem()
      }
    },

    // Auto-rotate methods
    startAutoRotateIfEnabled() {
      if (this.autoRotate) {
        this.startAutoRotate()
      }
    },

    startAutoRotate() {
      if (this.autoRotateId) return // Already running

      this.lastAutoRotateTime = performance.now()
      this.autoRotateId = requestAnimationFrame(this.updateAutoRotate)
    },

    stopAutoRotate() {
      if (this.autoRotateId) {
        cancelAnimationFrame(this.autoRotateId)
        this.autoRotateId = null
      }
    },

    updateAutoRotate(timestamp) {
      const elapsed = timestamp - this.lastAutoRotateTime
      this.lastAutoRotateTime = timestamp

      // Calculate rotation amount based on speed (degrees per second)
      const rotationAmount = (this.autoRotateSpeed * elapsed) / 1000

      // Apply rotation in the specified direction
      this.currentAngle += this.autoRotateClockwise ? rotationAmount : -rotationAmount

      if (this.$refs.wheel) {
        this.$refs.wheel.style.transform = `rotateY(${this.currentAngle}deg)`
      }

      // Continue auto-rotation
      this.autoRotateId = requestAnimationFrame(this.updateAutoRotate)
    },
  },
}
</script>
