<template>
  <div class="SectionPlan" ref="sectionPlan"
       @mousemove="onBuildingPlanMouseMove($event)"
       @mousedown="onBuildingPlanMouseDown($event)">
    <Table v-show="activeMode === modes.newTableTool" ref="newTable" :table="newTable"></Table>
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
          this.$refs.newTable.setInvalid(this.isActiveTableOverlapping(this.$refs.newTable))
        } else if (this.isDraggingMode(event)) {
          this.activeTable.setCoordinates({
            cx: event.clientX - this.left,
            cy: event.clientY - this.top
          })
          this.activeTable.setInvalid(this.isActiveTableOverlapping())
        }
      },
      onBuildingPlanMouseDown() {
        if (this.activeMode === this.modes.newTableTool) {
          this.$emit('addTable', { coordinates: this.newTable.coordinates })
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
        table.component = component;
      },
      setActiveTable(table) {
        if (this.activeTable) this.activeTable.makeInactive()
        this.activeTable = table
        if (table) table.makeActive()
        this.$emit('setActiveTable', table)
      },
      isDraggingMode(event) {
        return this.activeMode === this.modes.pointerTool &&
          this.activeTable &&
          event.which === 1
      },
      isActiveTableOverlapping(testableTable = this.activeTable) {
        const  testableTableBoundaries = testableTable.boundaries

        this.tables.forEach((table) => {
          const tableBoundaries = table.component.boundaries
          if (
            testableTableBoundaries.top > tableBoundaries.bottom ||
            testableTableBoundaries.right < tableBoundaries.left ||
            testableTableBoundaries.bottom < tableBoundaries.top ||
            testableTableBoundaries.left > tableBoundaries.right
          ) {
            console.log('overlap')
            return true
          }
        })
        console.log('does not overlap')
        return false
      }
    },
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
