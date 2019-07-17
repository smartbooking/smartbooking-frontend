<template>
  <canvas
    @mousedown="emitClick($event)"
    :width="size"
    :height="size"
    ref="canvas"
    :style="{ top: top, left: left, 'z-index': zIndex }"
  />
</template>

<script>
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
      radius: 20,
      size: 40,
      activeColor: 'green',
      inactiveColor: 'black',
      invalidColor: 'red',
      id: null,
      intersections: {}
    }
  },
  computed: {
    top() {
      return `${this.table.coordinates.cy - this.radius - 5}px`
    },
    left() {
      return `${this.table.coordinates.cx - this.radius - 5}px`
    },
    boundaries() {
      return {
        top: this.table.coordinates.cy - this.size / 2,
        right: this.table.coordinates.cx + this.size / 2,
        bottom: this.table.coordinates.cy + this.size / 2,
        left: this.table.coordinates.cx - this.size / 2
      }
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
      if (this.isInvalid) return this.invalidColor
      if (this.isActive) return this.activeColor

      return this.inactiveColor
    },
    isInvalid() {
      return Object.keys(this.intersections).length
    }
  },
  mounted() {
    this.drawingContext = this.$refs.canvas.getContext('2d')
    this.id = Math.random()
    this.redraw()
    this.$emit('created', this.table, this)
  },
  methods: {
    makeActive() {
      this.isActive = true
      this.redraw()
    },
    makeInactive() {
      this.isActive = false
      this.redraw()
    },
    emitClick($event) {
      this.$emit('click', this.table, $event)
    },
    redraw() {
      this.drawingContext.clearRect(0, 0, this.size, this.size)
      this.drawingContext.beginPath()
      this.drawingContext.arc(
        this.size / 2,
        this.size / 2,
        this.radius,
        0,
        2 * Math.PI
      )
      this.drawingContext.closePath()
      this.drawingContext.strokeStyle = this.strokeColor
      this.drawingContext.fillStyle = 'white'
      this.drawingContext.fill()
      this.drawingContext.textAlign = 'center'
      this.drawingContext.fillStyle = 'black'
      this.drawingContext.fillText(
        this.table.seatsCount,
        this.size / 2,
        this.size / 2
      )
      this.drawingContext.stroke()
    },
    setCoordinates({ cx, cy }) {
      this.table.coordinates.cx = cx
      this.table.coordinates.cy = cy
    },
    trackIntersection(anotherTable, isIntersecting) {
      if (isIntersecting) {
        this.$set(this.intersections, anotherTable.component.id, true)
      }
      else {
        this.$delete(this.intersections, anotherTable.component.id)
      }
    }
  },
  watch: {
    'table.seatsCount'() {
      this.redraw()
    },
    isInvalid() {
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
