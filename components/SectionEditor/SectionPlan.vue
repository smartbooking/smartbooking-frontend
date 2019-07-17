<template>
  <div class="SectionPlan" ref="sectionPlan"
       @mousemove="onBuildingPlanMouseMove($event)"
       @mousedown="onBuildingPlanMouseDown($event)">
    <Table v-show="activeMode === modes.newTableTool" :table="newTable" @created="setTableComponent"></Table>
    <Table v-for="table in tables" :key="table.id" :table="table" @click="onTableClick" @created="setTableComponent"></Table>
  </div>
</template>

<script>
  import Table from '~/components/SectionEditor/Table.vue'

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
          component: null
        },
        activeTable: null,
        activeMode: null,
        modes: {
          pointerTool: 'POINTER',
          newTableTool: 'NEW_TABLE'
        },
        left: 0,
        top: 0,
        isValid: true
      }
    },
    mounted() {
      const rect = this.$refs.sectionPlan.getBoundingClientRect()
      this.left = rect.left
      this.top = rect.top
    },
    methods: {
      onBuildingPlanMouseMove(event) {
        if (this.activeMode === this.modes.newTableTool) {
          this.newTable.coordinates = {
            cx: event.clientX - this.left,
            cy: event.clientY - this.top
          }
          this.testTableIntersection(this.newTable)
        } else if (this.isDraggingMode(event)) {
          this.activeTable.component.setCoordinates({
            cx: event.clientX - this.left,
            cy: event.clientY - this.top
          })
          this.testTableIntersection(this.activeTable)
        }
      },
      onBuildingPlanMouseDown() {
        if (this.activeMode === this.modes.newTableTool) {
          this.$emit('addTable', {
            coordinates: this.newTable.coordinates,
            seatsCount: this.newTable.seatsCount
          })
        } else {
          this.setActiveTable(null)
        }
      },
      setActiveTool(tool) {
        this.setActiveTable(null)
        this.activeMode = this.modes[tool.tool]
        this.newTable.coordinates.cx = -20
        this.newTable.coordinates.cy = -20
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
      testTableIntersection(testableTable = this.activeTable) {
        if (!testableTable) return false

        let result = false
        const testableTableBoundaries = testableTable.component.boundaries

        for (let table of this.tables) {
          if (table === testableTable) continue

          const tableBoundaries = table.component.boundaries

          if (Math.max(testableTableBoundaries.left, tableBoundaries.left) < Math.min(testableTableBoundaries.right, tableBoundaries.right) &&
            Math.max(testableTableBoundaries.top, tableBoundaries.top) < Math.min(testableTableBoundaries.bottom, tableBoundaries.bottom)) {
            testableTable.component.trackIntersection(table, true)
            table.component.trackIntersection(testableTable, true)
            result = true
          }
          else {
            testableTable.component.trackIntersection(table, false)
            table.component.trackIntersection(testableTable, false)
          }
        }

        this.$emit('onValidationResultChange', !result)

        return result
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
