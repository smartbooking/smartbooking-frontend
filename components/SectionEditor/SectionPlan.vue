<template>
  <div class="SectionPlan" ref="sectionPlan"
       @mousemove="onBuildingPlanMouseMove($event)"
       @mousedown="onBuildingPlanMouseDown($event)"
  >
    <Table v-show="activeMode === modes.newTableTool" :table="newTable" @created="setTableComponent"></Table>
    <Table v-for="table in tables" :key="table.id" :table="table" @click="onTableClick" @created="setTableComponent" @onValidationResultChange="$emit('onValidationResultChange', arguments[0])"></Table>
  </div>
</template>

<script>
  import { throttle } from 'lodash'
  import Table from '~/components/SectionEditor/Table.vue'

  const MOVE_TRESHOLD = 5

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
          coordinates: {
            cx: -20,
            cy: -20
          },
          seatsCount: 4,
          component: {},
        },
        activeTable: null,
        activeMode: null,
        modes: {
          pointerTool: 'POINTER',
          newTableTool: 'NEW_TABLE'
        },
        boundingClientRect: null,
        throttleTableIntersectionTest: throttle(this.setTableIntersection, 100),
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
        if (event.which == 8 &&
          this.activeTable &&
          this.activeMode == this.modes.pointerTool &&
          event.target.tagName == 'BODY'
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

        const newCoordinatesInvalid = (newCoordinates, radius) => {
          return newCoordinates.cx < radius ||
            newCoordinates.cx > this.boundingClientRect.width - radius ||
            newCoordinates.cy < radius ||
            newCoordinates.cy > this.boundingClientRect.height - radius
        }

        if (this.activeMode === this.modes.newTableTool) {
          const newCoordinates = {
            cx: Math.round((event.clientX - this.boundingClientRect.left) / MOVE_TRESHOLD) * MOVE_TRESHOLD,
            cy: Math.round((event.clientY - this.boundingClientRect.top) / MOVE_TRESHOLD) * MOVE_TRESHOLD
          }

          if (dragTresholdPassed(this.newTable.component.coordinates, newCoordinates) &&
            !newCoordinatesInvalid(newCoordinates, this.newTable.component.radius)
          ) {
            this.newTable.component.setCoordinates(newCoordinates)
            this.throttleTableIntersectionTest(this.newTable)
          }
        } else if (this.isDraggingMode(event)) {
          const newCoordinates = {
            cx: Math.round((event.clientX - this.boundingClientRect.left) / MOVE_TRESHOLD) * MOVE_TRESHOLD,
            cy: Math.round((event.clientY - this.boundingClientRect.top) / MOVE_TRESHOLD) * MOVE_TRESHOLD
          }

          if (dragTresholdPassed(this.activeTable.component.coordinates, newCoordinates) &&
            !newCoordinatesInvalid(newCoordinates, this.activeTable.component.radius)
          ) {
            this.activeTable.component.setCoordinates(newCoordinates)
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

        if (this.activeMode == this.modes.newTableTool) {
          this.setActiveTable(this.newTable)
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

        const testableTableBoundaries = testableTable.component.boundaries
        const testableTables = this.tables.filter(t => t.component)

        for (let table of testableTables) {
          if (table === testableTable) continue
          if (forcedValue !== null) {
            testableTable.component.trackIntersection(table, forcedValue)
            table.component.trackIntersection(testableTable, forcedValue)
            continue
          }

          const tableBoundaries = table.component.boundaries

          const intersects = Math.max(testableTableBoundaries.left, tableBoundaries.left) < Math.min(testableTableBoundaries.right, tableBoundaries.right) &&
            Math.max(testableTableBoundaries.top, tableBoundaries.top) < Math.min(testableTableBoundaries.bottom, tableBoundaries.bottom)

          testableTable.component.trackIntersection(table, intersects)
          table.component.trackIntersection(testableTable, intersects)
        }

        this.$emit(
          'onValidationResultChange',
          !testableTables.some(el => el.component.isInvalid)
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
