<template>
  <div
    class="min-h-screen flex flex-col bg-gradient-to-br from-gray-800 to-gray-900 text-white p-8"
  >
    <header class="mb-8 text-center">
      <h1 class="text-3xl font-bold mb-4">3D Carousel Component</h1>
      <p class="text-gray-300">Interactive 3D carousel with auto-rotation and inertial movement</p>
    </header>

    <div class="flex-1 flex flex-col items-center">
      <!-- Carousel Container -->
      <div class="w-full max-w-4xl h-96 mb-8">
        <Carousel3D
          :items="carouselItems"
          :item-width="itemWidth"
          :item-height="itemHeight"
          :radius="radius"
          :perspective="perspective"
          :sensitivity="sensitivity"
          :friction="friction"
          :pulse-scale="pulseScale"
          :pulse-duration="300"
          :show-controls="showControls"
          :info-text="'3D Wheel with Spokes - Drag to rotate or use buttons/arrow keys'"
          :rotation-step="rotationStep"
          :auto-rotate="autoRotate"
          :auto-rotate-speed="autoRotateSpeed"
          :auto-rotate-clockwise="autoRotateClockwise"
        />
      </div>

      <!-- Configuration Form -->
      <div class="w-full max-w-3xl mb-8 bg-gray-700 rounded-lg p-4">
        <h2 class="text-xl font-bold mb-3">Configuration</h2>

        <div class="grid grid-cols-2 sm:grid-cols-3 md:grid-cols-4 gap-2">
          <div class="flex flex-col">
            <label class="text-xs mb-1">Items</label>
            <input
              type="number"
              v-model.number="numItems"
              min="3"
              max="20"
              class="w-full px-2 py-1 text-sm rounded bg-gray-800 text-white border border-gray-600"
            />
          </div>

          <div class="flex flex-col">
            <label class="text-xs mb-1">Width (px)</label>
            <input
              type="number"
              v-model.number="itemWidth"
              min="50"
              max="500"
              class="w-full px-2 py-1 text-sm rounded bg-gray-800 text-white border border-gray-600"
            />
          </div>

          <div class="flex flex-col">
            <label class="text-xs mb-1">Height (px)</label>
            <input
              type="number"
              v-model.number="itemHeight"
              min="50"
              max="500"
              class="w-full px-2 py-1 text-sm rounded bg-gray-800 text-white border border-gray-600"
            />
          </div>

          <div class="flex flex-col">
            <label class="text-xs mb-1">Radius (px)</label>
            <input
              type="number"
              v-model.number="radius"
              min="50"
              max="500"
              class="w-full px-2 py-1 text-sm rounded bg-gray-800 text-white border border-gray-600"
            />
          </div>

          <div class="flex flex-col">
            <label class="text-xs mb-1">Perspective</label>
            <input
              type="number"
              v-model.number="perspective"
              min="500"
              max="2000"
              class="w-full px-2 py-1 text-sm rounded bg-gray-800 text-white border border-gray-600"
            />
          </div>

          <div class="flex flex-col">
            <label class="text-xs mb-1">Sensitivity</label>
            <input
              type="number"
              v-model.number="sensitivity"
              min="0.1"
              max="2"
              step="0.1"
              class="w-full px-2 py-1 text-sm rounded bg-gray-800 text-white border border-gray-600"
            />
          </div>

          <div class="flex flex-col">
            <label class="text-xs mb-1">Friction</label>
            <input
              type="number"
              v-model.number="friction"
              min="0.8"
              max="0.99"
              step="0.01"
              class="w-full px-2 py-1 text-sm rounded bg-gray-800 text-white border border-gray-600"
            />
          </div>

          <div class="flex flex-col">
            <label class="text-xs mb-1">Pulse Scale</label>
            <input
              type="number"
              v-model.number="pulseScale"
              min="1"
              max="1.2"
              step="0.01"
              class="w-full px-2 py-1 text-sm rounded bg-gray-800 text-white border border-gray-600"
            />
          </div>

          <div class="flex flex-col">
            <label class="text-xs mb-1">Rotation Step</label>
            <input
              type="number"
              v-model.number="rotationStep"
              min="15"
              max="90"
              step="15"
              class="w-full px-2 py-1 text-sm rounded bg-gray-800 text-white border border-gray-600"
            />
          </div>

          <div class="flex items-center">
            <input type="checkbox" id="showControls" v-model="showControls" class="mr-2" />
            <label for="showControls" class="text-xs">Controls</label>
          </div>

          <div class="flex items-center">
            <input type="checkbox" id="autoRotate" v-model="autoRotate" class="mr-2" />
            <label for="autoRotate" class="text-xs">Auto Rotate</label>
          </div>

          <div class="flex flex-col">
            <label class="text-xs mb-1">Rotate Speed</label>
            <input
              type="number"
              v-model.number="autoRotateSpeed"
              min="1"
              max="60"
              step="1"
              class="w-full px-2 py-1 text-sm rounded bg-gray-800 text-white border border-gray-600"
              :disabled="!autoRotate"
            />
          </div>

          <div class="col-span-2 flex items-center space-x-4">
            <div>
              <input
                type="radio"
                id="counterClockwise"
                v-model="autoRotateClockwise"
                :value="false"
                class="mr-1"
                :disabled="!autoRotate"
              />
              <label for="counterClockwise" class="text-xs mr-2">Counter-CW</label>

              <input
                type="radio"
                id="clockwise"
                v-model="autoRotateClockwise"
                :value="true"
                class="mr-1"
                :disabled="!autoRotate"
              />
              <label for="clockwise" class="text-xs">Clockwise</label>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import Carousel3D from '@/components/Carousel3D.vue'

export default {
  name: 'App',
  components: {
    Carousel3D,
  },
  data() {
    return {
      // Configuration options
      numItems: 12,
      itemWidth: 240,
      itemHeight: 140,
      radius: 120,
      perspective: 1200,
      sensitivity: 0.5,
      friction: 0.95,
      pulseScale: 1.02,
      rotationStep: 45,
      showControls: true,
      autoRotate: false,
      autoRotateSpeed: 15,
      autoRotateClockwise: false,
    }
  },
  computed: {
    carouselItems() {
      // Generate items based on numItems using Lorem Picsum
      return Array.from({ length: this.numItems }, (_, i) => ({
        // Using Lorem Picsum for random images
        image: `https://picsum.photos/id/${(i % 30) + 10}/${this.itemWidth}/${this.itemHeight}`,
        alt: `Item ${i + 1}`,
      }))
    },
  },
}
</script>
