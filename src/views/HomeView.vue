<template>
  <div id="app">
    <div class="drawing-area" @mousedown="startDrawing">
      <svg v-if="points.length > 1" class="svg-drawing">
        <polyline :points="pointsString" stroke="black" fill="none" />
      </svg>

      <svg class="svg-drawing">
        <polyline
          v-for="(shape, index) in this.svgpoints"
          :key="index"
          :points="shape"
          stroke="black"
          fill="none"
        />
      </svg>
      <div
        v-for="(shape, index) in shapes"
        :key="index"
        class="shape"
        :style="{
          left: shape.left + 'px',
          top: shape.top + 'px',
          width: shape.width + 'px',
          height: shape.height + 'px'
        }"
      ></div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      isDrawing: false,
      points: [],
      shapes: [],
      svgPoints: [],
      closedCercleAmount: 0
    }
  },
  computed: {
    pointsString() {
      return this.points.map((point) => `${point.x},${point.y}`).join(' ')
    }
  },
  methods: {
    startDrawing(event) {
      this.isDrawing = true
      this.points = [{ x: event.offsetX, y: event.offsetY }]

      window.addEventListener('mousemove', this.draw)
      window.addEventListener('mouseup', this.stopDrawing)
    },
    draw(event) {
      if (!this.isDrawing) return

      const newPoint = { x: event.offsetX, y: event.offsetY }
      this.points.push(newPoint)

      if (this.checkIfClosedShape(newPoint)) {
        this.stopDrawing()
        this.createShape()
      }
    },
    stopDrawing() {
      this.isDrawing = false
      window.removeEventListener('mousemove', this.draw)
      window.removeEventListener('mouseup', this.stopDrawing)
    },
    checkIfClosedShape(newPoint) {
      if (this.points.length < 10) return false

      const firstPoint = this.points[0]
      const distance = Math.sqrt(
        Math.pow(newPoint.x - firstPoint.x, 2) + Math.pow(newPoint.y - firstPoint.y, 2)
      )
      if (distance < 5) {
        this.closedCercleAmount += 1
      }
      return distance < 5
    },
    createShape() {
      /*
      const minX = Math.min(...this.points.map((point) => point.x))
      const minY = Math.min(...this.points.map((point) => point.y))
      const maxX = Math.max(...this.points.map((point) => point.x))
      const maxY = Math.max(...this.points.map((point) => point.y))

      const shape = {
        left: minX,
        top: minY,
        width: maxX - minX,
        height: maxY - minY
      }

      this.shapes.push(shape)
      */
      const svgShape = this.points.map((point) => `${point.x},${point.y}`).join(' ')
      this.svgPoints.push(svgShape)
      this.points = []
    }
  }
}
</script>

<style>
.drawing-area {
  width: 100%;
  height: 500px;
  background-color: #f0f0f0;
  position: relative;
}

.svg-drawing {
  position: absolute;
  width: 100%;
  height: 100%;
  pointer-events: none; /* Verhindert, dass das SVG das Zeichnen blockiert */
}

.shape {
  position: absolute;
  border: 2px solid #000;
  background-color: rgba(0, 0, 255, 0.1);
}
</style>
