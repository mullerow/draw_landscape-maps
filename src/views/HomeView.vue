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
      occupiedCoordinates: [],
      hoverIndex: null
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
      // das gehoverte polygon/SVG soll in den vordergrund gerückt werden, deshalb wird es nochmal an das ende der liste angefügt

      this.hoverIndex = index
      const hoveredShape = this.svgPoints.splice(index, 1)[0]
      this.svgPoints.push(hoveredShape)
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
      const startX = event.offsetX
      const startY = event.offsetY
      this.points = [{ x: event.offsetX, y: event.offsetY }]

      window.addEventListener('mousemove', this.draw)
      window.addEventListener('mouseup', this.stopDrawing)
      const occupiedArea = this.getOccupiedShapeIndex(startX, startY)

      if (occupiedArea !== null) {
        console.log(
          `Der Startpunkt ist bereits belegt von Shape mit Index: ${occupiedArea.shapeIndex}`
        )
      } else {
        console.log('Der Startpunkt ist frei.')
      }
    },
    getOccupiedShapeIndex(x, y) {
      const occupied = this.occupiedCoordinates.find(
        (occupied) => occupied.x === x && occupied.y === y
      )
      return occupied ? occupied : null
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
      this.points = this.removeDuplicatePoints()
      const cleanedPoints = this.removeOccupiedPoints()

      // Nur wenn es nach der Bereinigung noch Punkte gibt, den Scanline-Algorithmus anwenden
      if (cleanedPoints.length > 0) {
        this.points = cleanedPoints // Aktualisiere this.points auf die bereinigten Punkte
        this.fillOcupiedAreas() /////// Scanline-Algorithmus

        const svgShape = {
          points: cleanedPoints.map((point) => `${point.x},${point.y}`).join(' '),
          fill: 'lightgrey',
          stroke: 'black'
        }

        this.svgPoints.push(svgShape)
      }

      console.log('this.occupiedCoordinates', this.occupiedCoordinates)
      this.points = []
    },

    removeOccupiedPoints() {
      return this.points.filter(
        (point) => !this.isPointOccupied(Math.round(point.x), Math.round(point.y))
      )
    },

    isPointOccupied(x, y) {
      return this.occupiedCoordinates.some((coord) => coord.x === x && coord.y === y)
    },
    fillOcupiedAreas() {
      const { minY, maxY } = this.calculateBoundingBox()
      const shapeIndex = this.svgPoints.length

      for (let y = minY; y <= maxY; y++) {
        let contactPoints = []

        for (let i = 0; i < this.points.length; i++) {
          let j = (i + 1) % this.points.length
          const xi = this.points[i].x
          const yi = this.points[i].y
          const xj = this.points[j].x
          const yj = this.points[j].y
          // Vermeide horizontale Kanten oder Kanten, die auf der Scanlinie liegen
          if (yi === y && yj === y) {
            continue
          }

          if ((yi < y && yj >= y) || (yj < y && yi >= y)) {
            const ratio = (y - yi) / (yj - yi)
            const contactPointX = xi + ratio * (xj - xi)
            contactPoints.push(contactPointX)
          }
        }
        // muss von links nach rechts (aufsteigendes x sortiert werden, damit die schnittpaare zusammen sind)
        contactPoints.sort((a, b) => a - b)
        for (let k = 0; k < contactPoints.length; k += 2) {
          const xStart = Math.ceil(contactPoints[k])
          const xEnd = Math.floor(contactPoints[k + 1])
          for (let x = xStart; x <= xEnd; x++) {
            if (!this.occupiedCoordinates.includes(`${x},${y}`)) {
              this.occupiedCoordinates.push({ x, y, shapeIndex }) //(`${x},${y}`)
            }
          }
        }
      }
    },
    // die boundingbox limitiert den bereich der berechnet werden muss auf die maximalen ausdehnungen des shapes um unötige recharbeiten zu vermeiden
    calculateBoundingBox() {
      let minY = Math.min(...this.points.map((p) => p.y))
      let maxY = Math.max(...this.points.map((p) => p.y))
      return { minY, maxY }
    },
    removeDuplicatePoints() {
      const map = new Map()
      this.points.forEach((point) => {
        const key = `${point.x},${point.y}`
        if (!map.has(key)) {
          map.set(key, point)
        }
      })
      return Array.from(map.values())
    }
  }
}
</script>

<style>
html {
  background-color: #5c5c5c;
}
.drawing-area {
  width: 100%;
  height: 500px;
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
  background-color: #afaeae;
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
