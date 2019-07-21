<template>
  <div class="Wrapper">
    <v-container>
      <v-layout row wrap>
        <v-flex xs12>
          <Toolbox ref="toolbox" @setActiveTool="setActiveTool"></Toolbox>
        </v-flex>
        <v-flex xs9 class="SectionPlan">
          <SectionPlan ref="sectionPlan" @addTable="addTable"
                       @removeTable="removeTable" :tables="tables"
                       @setActiveTable="setActiveTable"
                       @onValidationResultChange="setValidationResult">
          </SectionPlan>
        </v-flex>
        <v-flex xs3 class="ActiveTablePreferences">
          <v-layout row wrap v-if="activeTable && sectionPlan.activeMode != sectionPlan.modes.newTableTool">
            <v-flex xs6>
              <v-text-field label="CY" v-bind:value="activeTable.coordinates.cy"
                            type="number" readonly/>
            </v-flex>
            <v-flex xs6>
              <v-text-field label="CX" v-bind:value="activeTable.coordinates.cx"
                            type="number" readonly/>
            </v-flex>
            <v-flex xs12>
              <v-text-field label="Seats Count" v-model="activeTable.seatsCount"
                            type="number"/>
            </v-flex>
          </v-layout>
          <span v-else>Select the table to edit its properties</span>
        </v-flex>
        <v-flex xs12>
          <div>Valid: {{validationResult}}</div>
          <v-btn block color="success" :disabled="!validationResult"
                 @click="saveConfiguration">Save
          </v-btn>
        </v-flex>
      </v-layout>
    </v-container>
  </div>
</template>
<script>
  import Toolbox from '~/components/SectionEditor/Toolbox.vue'
  import SectionPlan from '~/components/SectionEditor/SectionPlan.vue'

  const LOCAL_STORAGE_KEY = 'section-plan'

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
        return this.$refs.sectionPlan
      }
    },
    created() {
      const savedData = localStorage.getItem(LOCAL_STORAGE_KEY)

      if (savedData) this.tables = JSON.parse(savedData)
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
      removeTable(tableData) {
        this.$refs.sectionPlan.setActiveTable(null)
        this.$delete(this.tables, this.tables.indexOf(tableData));
      },
      setValidationResult(result) {
        this.validationResult = result
      },
      saveConfiguration() {
        const data = this.tables.map(table => {
          return {
            coordinates: table.coordinates,
            seatsCount: table.seatsCount
          }
        })

        localStorage.setItem(LOCAL_STORAGE_KEY, JSON.stringify(data))
      }
    }
  }
</script>

<style scoped lang="scss">
  .layout {
    height: 100%;
  }
  .Wrapper {
    width: 100%;
    height: 100%;
    display: flex;
    flex-direction: column;
    border: 1px solid white;
  }
  .SectionPlan {
    height: 100%
  }
  .ActiveTablePreferences {
    padding: 10px;
  }

  button {
    color: black;
  }
</style>
