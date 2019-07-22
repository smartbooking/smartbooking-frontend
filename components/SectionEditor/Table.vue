<template>
  <canvas
    @mousedown="emitClick($event)"
    ref="canvas"
    :style="{ top: top, left: left, 'z-index': zIndex }"
  />
</template>

<script>
const MAX_SEATS_COUNT = 8

export default {
  props: {
    table: {
      type: Object,
      required: true,
    }
  },
  data() {
    return {
      isActive: false,
      drawingContext: null,
      colors: {
        active: 'green',
        inactive: 'black',
        invalid: 'red'
      },
      id: null,
      intersections: {}
    }
  },
  computed: {
    radius() {
      return Math.ceil((Math.sqrt(Math.min(this.table.seatsCount, 20)) || 1) * 20)
    },
    size() {
      return this.radius * 2
    },
    top() {
      return `${this.table.coordinates.cy - this.radius}px`
    },
    left() {
      return `${this.table.coordinates.cx - this.radius}px`
    },
    coordinates() {
      return this.table.coordinates
    },
    width() {
      return this.size
    },
    height() {
      return this.size
    },
    zIndex() {
      return this.isActive ? 1 : 0
    },
    strokeColor() {
      if (this.isInvalid) return this.colors.invalid
      if (this.isActive) return this.colors.active

      return this.colors.inactive
    },
    isInvalid() {
      return Object.keys(this.intersections).length ||
        !this.table.seatsCount ||
        this.table.seatsCount > MAX_SEATS_COUNT
    }
  },
  mounted() {
    this.id = Math.random()
    this.drawingContext = this.$refs.canvas.getContext('2d')
    this.redraw()
    this.$emit('created', this.table, this)
  },
  methods: {
    makeActive() {
      this.isActive = true
    },
    makeInactive() {
      this.isActive = false
    },
    emitClick($event) {
      this.$emit('click', this.table, $event)
    },
    redraw() {
      this.drawingContext.clearRect(
        0,
        0,
        this.width,
        this.height
      )
      this.$refs.canvas.width = this.width
      this.$refs.canvas.height = this.height
      this.drawingContext.beginPath()
      this.drawingContext.fillStyle = 'black'
      this.drawingContext.strokeStyle = this.strokeColor
      this.drawingContext.arc(this.radius, this.radius, this.radius, 0, 2 * Math.PI)
      this.drawingContext.stroke()
      this.drawingContext.fillStyle = 'white'
      this.drawingContext.fill()
      this.drawingContext.textAlign = 'center'
      this.drawingContext.fillStyle = 'black'
      this.drawingContext.font = "16px serif"
      this.drawingContext.fillText(
        this.table.seatsCount,
        this.radius,
        this.radius
      )

      this.drawingContext.font = "9px serif"
      this.drawingContext.fillText(
        'seats',
        this.size / 2,
        this.size / 2 + 12
      )
      this.drawingContext.stroke()
      this.drawingContext.closePath()
    },
    setCoordinates({ cx, cy }) {
      this.table.coordinates.cx = cx
      this.table.coordinates.cy = cy
    },
    trackIntersection(anotherTable, isIntersecting) {
      if (isIntersecting) {
        this.$set(this.intersections, anotherTable.component.id, true)
      } else {
        this.$delete(this.intersections, anotherTable.component.id)
      }
    }
  },
  watch: {
    'table.seatsCount'() {
      this.redraw()
    },
    isInvalid(isNowInvalid) {
      this.redraw()
      this.$emit('onValidationResultChange', !isNowInvalid)
    },
    isActive() {
      this.redraw()
    }
  }
}
</script>

<style scoped lang="scss">
canvas {
  position: absolute;
  &:hover {
    cursor: pointer;
  }
}
</style>
