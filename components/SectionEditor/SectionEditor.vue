<template>
  <div class="Wrapper">
    <Toolbox ref="toolbox" @setActiveTool="setActiveTool"></Toolbox>
    <SectionPlan ref="sectionPlan" @addTable="addTable" :tables="tables" @setActiveTable="setActiveTable" @onValidationResultChange="setValidationResult">
    </SectionPlan>
    <span v-if="activeTable">
      {{ activeTable.component.boundaries }}
      <div>
        Seats Count: <input type="number" v-model="activeTable.seatsCount"/>
      </div>
    </span>
    Valid: {{validationResult}}
  </div>
</template>
<script>
import Toolbox from '~/components/SectionEditor/Toolbox.vue'
import SectionPlan from '~/components/SectionEditor/SectionPlan.vue'

export default {
  components: { Toolbox, SectionPlan },
  data() {
    return {
      tables: [],
      activeTable: null,
      validationResult: true
    }
  },
  computed: {
    sectionPlan() {
      return this.$refs.sectionPlan || {}
    }
  },
  methods: {
    setActiveTool(tool) {
      this.$refs.sectionPlan.setActiveTool(tool)
    },
    setActiveTable(activeTable) {
      this.activeTable = activeTable
    },
    addTable(tableData) {
      this.tables.push(tableData)
    },
    setValidationResult(result) {
      this.validationResult = result;
    }
  }
}
</script>

<style scoped lang="scss">
.Wrapper {
  width: 400px;
  height: 450px;
  display: flex;
  flex-direction: column;
}
  button {
    color: black;
  }
</style>
