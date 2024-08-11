<template>
  <div id="app">
    <div class="drawing-area" @mousedown="startDrawing" @mousemove="draw" @mouseup="stopDrawing">
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
    <div v-if="this.isDrawing">{{ this.points }}</div>
  </div>
</template>
<script>
export default {
  data() {
    return {
      isDrawing: false,
      points: [],
      shapes: []
    }
  },
  methods: {
    startDrawing(event) {
      this.isDrawing = true
      this.points = [{ x: event.offsetX, y: event.offsetY }]
      console.log('Zeichnen gestartet', event.offsetX, event.offsetY)
    },

    draw(event) {
      if (!this.isDrawing) return

      const newPoint = { x: event.offsetX, y: event.offsetY }
      this.points.push(newPoint)
      console.log('Zeichnen läuft', newPoint)
      if (this.checkIfClosedShape(newPoint)) {
        // this.stopDrawing()
        this.createShape()
      }
    },
    stopDrawing() {
      this.isDrawing = false
      this.points = []
    },
    checkIfClosedShape(newPoint) {
      if (this.points.length < 3) return false

      const firstPoint = this.points[0]
      const distance = Math.sqrt(
        Math.pow(newPoint.x - firstPoint.x, 2) + Math.pow(newPoint.y - firstPoint.y, 2)
      )
      console.log('abstand bis geschlossen', distance)
      return distance < 10 // Schwellenwert für die Erkennung eines geschlossenen Kreises
    },
    createShape() {
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

.shape {
  position: absolute;
  border: 2px solid #000;
  background-color: rgba(0, 0, 255, 0.1);
}
</style>
