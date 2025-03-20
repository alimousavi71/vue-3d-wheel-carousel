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
import { defineComponent, ref, onMounted, onBeforeUnmount, watch } from 'vue'

// Define item interface
interface CarouselItem {
  image: string
  alt?: string
}

export default defineComponent({
  name: 'Carousel3D',
  props: {
    // Items to display in the carousel
    items: {
      type: Array as () => CarouselItem[],
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
  setup(props, { emit }) {
    // Refs
    const wheel = ref<HTMLDivElement | null>(null)
    const currentAngle = ref(0)
    const isDragging = ref(false)
    const startX = ref(0)
    const lastX = ref(0)
    const startAngle = ref(0)
    const velocity = ref(0)
    const lastTimestamp = ref(0)
    const animationId = ref<number | null>(null)
    const autoRotateId = ref<number | null>(null)
    const lastAutoRotateTime = ref(0)

    // Methods
    const getItemTransform = (index: number): string => {
      const angle = (index / props.items.length) * 360

      return `
        rotateY(${angle}deg) 
        translateZ(-20px)
        translateX(${props.radius}px)
      `
    }

    const prevItem = () => {
      rotateWheel(props.rotationStep)
    }

    const nextItem = () => {
      rotateWheel(-props.rotationStep)
    }

    const applyMicroAnimation = () => {
      if (!wheel.value) return

      const cards = wheel.value.querySelectorAll('.absolute')

      cards.forEach((card) => {
        const currentTransform = (card as HTMLElement).style.transform

        // Apply a subtle pulse scale animation
        ;(card as HTMLElement).style.transform =
          `${currentTransform.split('scale')[0]} scale(${props.pulseScale})`

        // Reset after animation completes
        setTimeout(() => {
          ;(card as HTMLElement).style.transform = currentTransform
        }, props.pulseDuration)
      })
    }

    const rotateWheel = (degrees: number) => {
      // Stop auto-rotate temporarily
      stopAutoRotate()

      // Cancel any ongoing animation
      if (animationId.value) {
        cancelAnimationFrame(animationId.value)
        animationId.value = null
      }

      // Apply subtle micro-animation
      applyMicroAnimation()

      // Enable transition for smooth rotation
      if (wheel.value) {
        wheel.value.style.transition = 'transform 0.5s ease'
      }

      // Update angle
      currentAngle.value += degrees
      if (wheel.value) {
        wheel.value.style.transform = `rotateY(${currentAngle.value}deg)`
      }

      // Reset velocity and animation state
      velocity.value = 0
      lastTimestamp.value = 0

      // Reset transition after animation completes
      setTimeout(() => {
        if (wheel.value) {
          wheel.value.style.transition = 'none'
        }

        // Restart auto-rotate if it was enabled
        if (props.autoRotate) {
          startAutoRotate()
        }
      }, 500)
    }

    const startDrag = (e: MouseEvent) => {
      e.preventDefault()

      // Stop auto-rotate while dragging
      stopAutoRotate()

      // Stop any ongoing animation
      if (animationId.value) {
        cancelAnimationFrame(animationId.value)
        animationId.value = null
      }

      isDragging.value = true
      startX.value = e.clientX
      lastX.value = startX.value
      startAngle.value = currentAngle.value
      lastTimestamp.value = 0
      velocity.value = 0

      // Disable transition during drag
      if (wheel.value) {
        wheel.value.style.transition = 'none'
      }

      // Change cursor
      document.body.style.cursor = 'grabbing'
    }

    const onDrag = (e: MouseEvent) => {
      if (!isDragging.value) return

      // Calculate velocity
      const now = performance.now()
      const elapsed = now - lastTimestamp.value || 16
      const deltaX = e.clientX - lastX.value

      // Update velocity (scaled for rotation)
      velocity.value = (deltaX * 0.3) / elapsed

      // Calculate total rotation
      const totalDeltaX = e.clientX - startX.value
      const newAngle = startAngle.value + totalDeltaX * props.sensitivity

      // Update wheel rotation
      if (wheel.value) {
        wheel.value.style.transform = `rotateY(${newAngle}deg)`
      }
      currentAngle.value = newAngle

      // Store position for next calculation
      lastX.value = e.clientX
      lastTimestamp.value = now
    }

    const stopDrag = () => {
      if (!isDragging.value) return

      isDragging.value = false

      // Reset cursor
      document.body.style.cursor = 'default'

      // Start inertia animation
      lastTimestamp.value = 0
      animationId.value = requestAnimationFrame(applyInertia)

      // Restart auto-rotate after inertia if it was enabled
      if (props.autoRotate) {
        // Wait for inertia to finish before restarting auto-rotate
        const checkInertiaFinished = () => {
          if (animationId.value) {
            setTimeout(checkInertiaFinished, 100)
          } else {
            startAutoRotate()
          }
        }

        setTimeout(checkInertiaFinished, 100)
      }
    }

    const applyInertia = (timestamp: number) => {
      if (!lastTimestamp.value) {
        lastTimestamp.value = timestamp
        animationId.value = requestAnimationFrame(applyInertia)
        return
      }

      const elapsed = timestamp - lastTimestamp.value
      lastTimestamp.value = timestamp

      // Apply friction
      velocity.value *= props.friction

      // Update angle based on velocity
      currentAngle.value += (velocity.value * elapsed) / 16

      if (wheel.value) {
        wheel.value.style.transform = `rotateY(${currentAngle.value}deg)`
      }

      // Apply micro-animation at start of inertial movement
      if (Math.abs(velocity.value) > 1.5) {
        applyMicroAnimation()
      }

      // Continue animation or stop
      if (Math.abs(velocity.value) > 0.1) {
        animationId.value = requestAnimationFrame(applyInertia)
      } else {
        lastTimestamp.value = 0
        velocity.value = 0
        animationId.value = null
      }
    }

    const handleKeyDown = (e: KeyboardEvent) => {
      if (e.key === 'ArrowLeft') {
        prevItem()
      } else if (e.key === 'ArrowRight') {
        nextItem()
      }
    }

    const startAutoRotateIfEnabled = () => {
      if (props.autoRotate) {
        startAutoRotate()
      }
    }

    const startAutoRotate = () => {
      if (autoRotateId.value) return // Already running

      lastAutoRotateTime.value = performance.now()
      autoRotateId.value = requestAnimationFrame(updateAutoRotate)
    }

    const stopAutoRotate = () => {
      if (autoRotateId.value) {
        cancelAnimationFrame(autoRotateId.value)
        autoRotateId.value = null
      }
    }

    const updateAutoRotate = (timestamp: number) => {
      const elapsed = timestamp - lastAutoRotateTime.value
      lastAutoRotateTime.value = timestamp

      // Calculate rotation amount based on speed (degrees per second)
      const rotationAmount = (props.autoRotateSpeed * elapsed) / 1000

      // Apply rotation in the specified direction
      currentAngle.value += props.autoRotateClockwise ? rotationAmount : -rotationAmount

      if (wheel.value) {
        wheel.value.style.transform = `rotateY(${currentAngle.value}deg)`
      }

      // Continue auto-rotation
      autoRotateId.value = requestAnimationFrame(updateAutoRotate)
    }

    // Lifecycle hooks
    onMounted(() => {
      // Prevent browser's default drag behavior
      wheel.value?.addEventListener('dragstart', (e) => e.preventDefault())

      // Add event listeners for drag
      document.addEventListener('mousemove', onDrag)
      document.addEventListener('mouseup', stopDrag)
      document.addEventListener('mouseleave', stopDrag)

      // Add keyboard controls
      document.addEventListener('keydown', handleKeyDown)

      // Start auto-rotate if enabled
      startAutoRotateIfEnabled()
    })

    onBeforeUnmount(() => {
      // Clean up event listeners
      document.removeEventListener('mousemove', onDrag)
      document.removeEventListener('mouseup', stopDrag)
      document.removeEventListener('mouseleave', stopDrag)
      document.removeEventListener('keydown', handleKeyDown)

      // Cancel any ongoing animations
      if (animationId.value) {
        cancelAnimationFrame(animationId.value)
      }

      // Cancel auto-rotate
      stopAutoRotate()
    })

    // Watchers
    watch(
      () => props.autoRotate,
      (newValue) => {
        if (newValue) {
          startAutoRotate()
        } else {
          stopAutoRotate()
        }
      },
    )

    watch(
      () => props.autoRotateSpeed,
      () => {
        if (props.autoRotate) {
          // Restart auto-rotate with new speed
          stopAutoRotate()
          startAutoRotate()
        }
      },
    )

    return {
      wheel,
      getItemTransform,
      prevItem,
      nextItem,
      startDrag,
    }
  },
})
</script>
