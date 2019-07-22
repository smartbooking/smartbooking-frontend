<template>
  <div
    ref="sectionPlan"
    class="SectionPlan"
    @mousemove="onBuildingPlanMouseMove($event)"
    @mousedown="onBuildingPlanMouseDown($event)"
  >
    <Table
      v-show="activeMode === modes.newTableTool"
      :table="newTable"
      @created="setTableComponent"
    />
    <Table
      v-for="table in tables"
      :key="table.id"
      :table="table"
      @click="onTableClick"
      @created="setTableComponent"
      @onValidationResultChange="throttleValidationResultChange"
    />
  </div>
</template>

<script>
import { throttle, clone } from 'lodash'
import Table from '~/components/SectionEditor/Table.vue'

const MOVE_TRESHOLD = 5
const NEW_TABLE_INITIAL_COORDINATES = { cx: -100, cy: -100 }

export default {
  components: {
    Table
  },
  props: {
    tables: {
      type: Array,
      required: true
    }
  },
  data() {
    return {
      newTable: {
        coordinates: clone(NEW_TABLE_INITIAL_COORDINATES),
        seatsCount: 4,
        component: {}
      },
      activeTable: null,
      activeMode: null,
      modes: {
        pointerTool: 'POINTER',
        newTableTool: 'NEW_TABLE'
      },
      boundingClientRect: null,
      throttleTableIntersectionTest: throttle(this.setTableIntersection, 100),
      throttleValidationResultChange: throttle(this.recalculateValidationResult, 100)
    }
  },
  computed: {
    testableTables() {
      return this.tables.filter(t => t.component)
    }
  },
  mounted() {
    this.setBoundingClientRect()
    document.addEventListener('keydown', this.onKeyDown)
    document.addEventListener('scroll', this.setBoundingClientRect)
  },
  methods: {
    setBoundingClientRect() {
      this.boundingClientRect = this.$refs.sectionPlan.getBoundingClientRect()
    },
    onKeyDown(event) {
      if (
        event.which === 8 &&
        this.activeTable &&
        this.activeMode === this.modes.pointerTool &&
        event.target.tagName === 'BODY'
      ) {
        this.setTableIntersection(this.activeTable, false)
        this.$emit('removeTable', this.activeTable)
      }
    },
    onBuildingPlanMouseMove(event) {
      const dragTresholdPassed = (oldCoordinates, newCoordinates) => {
        return Math.abs(oldCoordinates.cx - newCoordinates.cx) >= MOVE_TRESHOLD ||
          Math.abs(oldCoordinates.cy - newCoordinates.cy) >= MOVE_TRESHOLD
      }

      const coercedNewCoordinates = (newCoordinates, radius) => {
        let coercedCx = newCoordinates.cx < radius ? radius : newCoordinates.cx
        coercedCx = coercedCx > this.boundingClientRect.width - radius ? this.boundingClientRect.width - radius : coercedCx

        let coercedCy = newCoordinates.cy < radius ? radius : newCoordinates.cy
        coercedCy = coercedCy > this.boundingClientRect.height - radius ? this.boundingClientRect.height - radius : coercedCy

        return { cx: coercedCx, cy: coercedCy }
      }

      if (this.activeMode === this.modes.newTableTool) {
        const newCoordinates = {
          cx: Math.round((event.clientX - this.boundingClientRect.left) / MOVE_TRESHOLD) * MOVE_TRESHOLD,
          cy: Math.round((event.clientY - this.boundingClientRect.top) / MOVE_TRESHOLD) * MOVE_TRESHOLD
        }

        if (dragTresholdPassed(this.newTable.component.coordinates, newCoordinates)) {
          this.newTable.component.setCoordinates(
            coercedNewCoordinates(
              newCoordinates,
              this.newTable.component.radius
            )
          )
          this.throttleTableIntersectionTest(this.newTable)
        }
      } else if (this.isDraggingMode(event)) {
        const newCoordinates = {
          cx: Math.round((event.clientX - this.boundingClientRect.left) / MOVE_TRESHOLD) * MOVE_TRESHOLD,
          cy: Math.round((event.clientY - this.boundingClientRect.top) / MOVE_TRESHOLD) * MOVE_TRESHOLD
        }

        if (dragTresholdPassed(this.activeTable.component.coordinates, newCoordinates)) {
          this.activeTable.component.setCoordinates(
            coercedNewCoordinates(
              newCoordinates,
              this.activeTable.component.radius
            )
          )
          this.throttleTableIntersectionTest(this.activeTable)
        }
      }
    },
    onBuildingPlanMouseDown(event) {
      event.preventDefault()
      event.stopPropagation()

      if (this.activeMode === this.modes.newTableTool) {
        this.setTableIntersection(this.newTable)
        if (this.newTable.component.isInvalid) return
        this.$emit('addTable', {
          coordinates: { cx: this.newTable.coordinates.cx, cy: this.newTable.coordinates.cy },
          seatsCount: this.newTable.seatsCount
        })
      } else {
        this.setActiveTable(null)
      }
    },
    setActiveTool(tool) {
      this.setActiveTable(null)
      this.activeMode = this.modes[tool.tool]

      if (this.activeMode === this.modes.newTableTool) {
        this.setActiveTable(this.newTable)
      } else if (this.activeMode === this.modes.pointerTool) {
        this.newTable.coordinates = clone(NEW_TABLE_INITIAL_COORDINATES)
        this.setTableIntersection(this.newTable, false)
      }
    },
    onTableClick(table, $event) {
      if (this.activeMode === this.modes.pointerTool) {
        $event.stopImmediatePropagation()
        $event.preventDefault()
        this.setActiveTable(table)
      }
    },
    setTableComponent(table, component) {
      table.component = component
      this.setTableIntersection(table)
    },
    setActiveTable(table) {
      if (this.activeTable) this.activeTable.component.makeInactive()
      this.activeTable = table
      if (table) table.component.makeActive()
      this.$emit('setActiveTable', table)
    },
    isDraggingMode(event) {
      return this.activeMode === this.modes.pointerTool &&
        this.activeTable &&
        event.which === 1
    },
    setTableIntersection(testableTable = this.activeTable, forcedValue = null) {
      if (!testableTable) return

      for (const table of this.testableTables) {
        if (table === testableTable) continue
        if (forcedValue !== null) {
          testableTable.component.trackIntersection(table, forcedValue)
          table.component.trackIntersection(testableTable, forcedValue)
          continue
        }

        const distanceBetweenCenters = Math.sqrt(
          Math.pow(testableTable.coordinates.cx - table.coordinates.cx, 2) +
            Math.pow(testableTable.coordinates.cy - table.coordinates.cy, 2)
        )

        const intersects =
          distanceBetweenCenters <
          table.component.radius + testableTable.component.radius

        testableTable.component.trackIntersection(table, intersects)
        table.component.trackIntersection(testableTable, intersects)
      }

      this.throttleValidationResultChange()
    },
    recalculateValidationResult() {
      this.$emit(
        'onValidationResultChange',
        !this.testableTables.some(el => el.component.isInvalid)
      )
    }
  }
}
</script>

<style scoped lang="scss">
.SectionPlan {
  height: 100%;
  background-color: white;
  overflow: hidden;
  position: relative;
}
</style>
