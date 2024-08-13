<template>
  <div id="app">
    <div class="drawing-area" @mousedown="startDrawing">
      <svg v-if="points.length > 1" class="svg-drawing">
        <polyline :points="pointsString" stroke="black" fill="none" />
      </svg>
      <svg class="svg-drawing-circle">
        <polyline
          v-for="(shape, index) in this.svgPoints"
          :key="index"
          :points="shape.points"
          stroke="black"
          :fill="shape.fill"
        />
      </svg>
    </div>
    <button @click="changeColor('90e0ef')">LEVEL 1</button>
    <button @click="changeColor('00b4d8')">LEVEL 2</button>
    <button @click="changeColor('0077b6')">LEVEL 3</button>
    <button @click="changeColor('03045e')">LEVEL 4</button>
    <input type="number" id="numberInput" value="1" />
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
      choosenShapeNumber: 0
    }
  },
  computed: {
    pointsString() {
      return this.points.map((point) => `${point.x},${point.y}`).join(' ')
    }
  },
  methods: {
    changeColor(color) {
      const input = document.getElementById('numberInput')
      this.choosenShapeNumber = input.value
      this.svgPoints[this.choosenShapeNumber - 1].fill = `#${color}`
    },
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
      return distance < 5
    },
    createShape() {
      const svgShape = {
        points: this.points.map((point) => `${point.x},${point.y}`).join(' '),
        fill: 'lightgrey'
      }
      this.svgPoints.push(svgShape)
      console.log('svgPoints:', this.svgPoints)
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
  pointer-events: none; /* Verhindert, dass das SVG das Zeichnen blockiert */
  width: 100%;
  height: 500px;
}
.svg-drawing-circle {
  width: 100%;
  height: 100%;
  background-color: #caf0f8;
}

.shape {
  position: absolute;
  border: 2px solid #000;
  background-color: rgba(0, 0, 255, 0.1);
}
</style>
