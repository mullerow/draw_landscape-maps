<template>
  <div id="app">
    <div class="drawing-area" @mousedown="startDrawing">
      <svg v-if="points.length > 1" class="svg-drawing">
        <polyline :points="pointsString" stroke="black" fill="none" />
      </svg>
      <svg class="svg-drawing-circle">
        <polyline
          :class="{ 'polyline-shape': true, 'polyline-hovered': hoverIndex === index }"
          v-for="(shape, index) in this.svgPoints"
          :key="index"
          :points="shape.points"
          :stroke="shape.stroke"
          :fill="shape.fill"
          @mouseenter="startHoverShape(index)"
          @mouseleave="endHoverShape(index)"
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
      choosenShapeNumber: 0,
      shapeColorBeforHover: '',
      occupiedCoordinates: []
    }
  },
  computed: {
    pointsString() {
      return this.points.map((point) => `${point.x},${point.y}`).join(' ')
    }
  },
  methods: {
    startHoverShape(index) {
      this.svgPoints[index].stroke = 'green'
      this.shapeColorBeforHover = this.svgPoints[index].fill
      this.svgPoints[index].fill = 'green'
      this.hoverIndex = index
    },
    endHoverShape(index) {
      this.svgPoints[index].stroke = 'black'
      this.svgPoints[index].fill = this.shapeColorBeforHover
      this.hoverIndex = null
    },
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
        fill: 'lightgrey',
        stroke: 'black'
      }
      this.fillOcupiedAreas()
      this.svgPoints.push(svgShape)
      console.log('svgPoints:', this.svgPoints)
      this.points = []
      /////// Scanline-Algorithmus
    },
    fillOcupiedAreas() {
      const { minX, minY, maxX, maxY } = this.calculateBoundingBox(this.points)
      console.log('boundingbox', minX, minY, maxX, maxY)
      for (let y = minY; y <= maxY; y++) {
        let contactPoints = []
        for (let i = 0; i < this.points.length; i++) {
          let j = (i + 1) % this.points.length
          const xi = this.points[i].x
          const yi = this.points[i].y
          const xj = this.points[j].x
          const yj = this.points[j].y

          if ((yi <= y && y < yj) || (yj <= y && y < yi)) {
            // überprüft, ob die scan-linie (y) eine kante des polygons schneidet
            const contactPointsX = Math.round(xi + ((y - yi) * (xj - xi)) / (yj - yi))
            contactPoints.push(contactPointsX)
          }
        }
        // muss von links nach rechts (aufsteigendes x sortiert werden, damit die schnittpaare zusammen sind)
        contactPoints.sort((a, b) => a - b)

        for (let k = 0; k < contactPoints.length; k += 2) {
          let startXcontactPoint = Math.ceil(contactPoints[k])
          let endXcontactPoint = Math.floor(contactPoints[k + 1])

          for (let x = startXcontactPoint; x <= endXcontactPoint; x++) {
            if (!this.occupiedCoordinates.includes(`${x},${y}`)) {
              this.occupiedCoordinates.push(`${x},${y}`)
            }
          }
        }
      }
    },
    calculateBoundingBox(points) {
      let minX = Math.min(...points.map((p) => p.x))
      let minY = Math.min(...points.map((p) => p.y))
      let maxX = Math.max(...points.map((p) => p.x))
      let maxY = Math.max(...points.map((p) => p.y))

      return { minX, minY, maxX, maxY }
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
.polyline-shape {
  stroke-width: 1;
  stroke: black;
  transition:
    stroke 0.3s,
    stroke-width 0.3s;
}
.polyline-hovered {
  stroke: red;

  stroke-width: 10;
}
</style>
