# Vue 3D Carousel Component

![Vue 3D Carousel Header](https://awebmaker.ir/vue-3d-wheel-carousel.png)

A highly customizable 3D carousel component for Vue.js applications with smooth animations, inertial scrolling, and auto-rotation features.

## Demo

[Live Demo](https://awebmaker.ir/vue-3d-wheel-carousel)

## Features

- ✨ True 3D carousel with perspective
- 🖱️ Interactive mouse drag with inertia effect
- ⌨️ Keyboard navigation (arrow keys)
- 🔄 Auto-rotation with adjustable speed
- 📱 Mobile-friendly
- 🎨 Customizable appearance

## How to Use

### Step 1: Download the Component

Download the `Carousel3D.vue` file from this repository.

### Step 2: Add to Your Project

Place the `Carousel3D.vue` file in your project's components directory.

### Step 3: Import and Use

```vue
<template>
  <div style="height: 400px">
    <Carousel3D :items="carouselItems" />
  </div>
</template>

<script>
import Carousel3D from './components/Carousel3D.vue'

export default {
  components: {
    Carousel3D,
  },
  data() {
    return {
      carouselItems: [
        { image: 'https://picsum.photos/id/10/180/240', alt: 'Image 1' },
        { image: 'https://picsum.photos/id/11/180/240', alt: 'Image 2' },
        { image: 'https://picsum.photos/id/12/180/240', alt: 'Image 3' },
        { image: 'https://picsum.photos/id/13/180/240', alt: 'Image 4' },
        { image: 'https://picsum.photos/id/14/180/240', alt: 'Image 5' },
        { image: 'https://picsum.photos/id/15/180/240', alt: 'Image 6' },
        { image: 'https://picsum.photos/id/16/180/240', alt: 'Image 7' },
        { image: 'https://picsum.photos/id/17/180/240', alt: 'Image 8' },
      ],
    }
  },
}
</script>
```

## Requirements

- Vue.js 3.x
- Tailwind CSS (for styling)

If you're not using Tailwind CSS, you can convert the Tailwind classes to regular CSS.

## Basic Properties

| Prop         | Type   | Default | Description                                        |
| ------------ | ------ | ------- | -------------------------------------------------- |
| `items`      | Array  | `[]`    | Array of objects with `image` and `alt` properties |
| `itemWidth`  | Number | `180`   | Width of each item in pixels                       |
| `itemHeight` | Number | `240`   | Height of each item in pixels                      |
| `radius`     | Number | `120`   | Distance from center to items                      |

## Adding Auto-Rotation

```vue
<template>
  <Carousel3D :items="carouselItems" :auto-rotate="true" :auto-rotate-speed="15" />
</template>
```

## Full Property List

| Prop                  | Type    | Default      | Description                                        |
| --------------------- | ------- | ------------ | -------------------------------------------------- |
| `items`               | Array   | `[]`         | Array of objects with `image` and `alt` properties |
| `itemWidth`           | Number  | `180`        | Width of each item in pixels                       |
| `itemHeight`          | Number  | `240`        | Height of each item in pixels                      |
| `radius`              | Number  | `120`        | Distance from center to items                      |
| `perspective`         | Number  | `1200`       | 3D perspective value                               |
| `sensitivity`         | Number  | `0.5`        | Mouse drag sensitivity                             |
| `friction`            | Number  | `0.95`       | Inertia friction factor (0-1)                      |
| `pulseScale`          | Number  | `1.02`       | Scale factor for micro-animation                   |
| `pulseDuration`       | Number  | `300`        | Duration of pulse animation in ms                  |
| `showControls`        | Boolean | `true`       | Show navigation buttons                            |
| `prevText`            | String  | `'Previous'` | Previous button text                               |
| `nextText`            | String  | `'Next'`     | Next button text                                   |
| `showInfo`            | Boolean | `true`       | Show information banner                            |
| `rotationStep`        | Number  | `45`         | Rotation angle for button navigation               |
| `autoRotate`          | Boolean | `false`      | Enable auto-rotation                               |
| `autoRotateSpeed`     | Number  | `10`         | Auto-rotation speed (degrees per second)           |
| `autoRotateClockwise` | Boolean | `false`      | Auto-rotation direction                            |

## Responsive Example

Adjust the carousel for different screen sizes:

```vue
<template>
  <Carousel3D
    :item-width="windowWidth < 768 ? 120 : 180"
    :item-height="windowWidth < 768 ? 160 : 240"
    :radius="windowWidth < 768 ? 80 : 120"
    :items="carouselItems"
  />
</template>

<script>
export default {
  data() {
    return {
      windowWidth: window.innerWidth,
      carouselItems: [
        /* ... */
      ],
    }
  },
  mounted() {
    window.addEventListener('resize', this.updateWidth)
  },
  beforeUnmount() {
    window.removeEventListener('resize', this.updateWidth)
  },
  methods: {
    updateWidth() {
      this.windowWidth = window.innerWidth
    },
  },
}
</script>
```

## Browser Support

Works in all modern browsers:

- Chrome/Edge (latest)
- Firefox (latest)
- Safari (latest)
- Mobile browsers (latest)

## License

[MIT License](LICENSE) - Feel free to use this component in your projects.

---

Feel free to modify this component to suit your needs! If you find it useful, consider giving this project a star on GitHub.
