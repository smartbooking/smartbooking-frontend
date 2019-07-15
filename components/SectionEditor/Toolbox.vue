<template>
  <div class="Toolbox">
    <ToolboxTool
      ref="pointerTool"
      :icon="'my_location'"
      :tool="'pointerTool'"
      @select="setActiveTool"
    >
    </ToolboxTool>
    <ToolboxTool
      ref="newTableTool"
      :icon="'all_out'"
      :tool="'newTableTool'"
      @select="setActiveTool"
    >
    </ToolboxTool>
  </div>
</template>

<script>
import ToolboxTool from '~/components/SectionEditor/ToolboxTool.vue'

export default {
  components: {
    ToolboxTool
  },
  data() {
    return {
      activeTool: null
    }
  },
  computed: {
    pointerTool() {
      return this.$refs.pointerTool
    },
    newTableTool() {
      return this.$refs.newTableTool
    }
  },
  mounted() {
    this.setActiveTool(this.$refs.pointerTool);
  },
  methods: {
    setActiveTool(tool) {
      if (this.activeTool) {
        this.activeTool.makeInactive()
      }
      this.activeTool = tool
      tool.makeActive()
      this.$emit('setActiveTool', tool)
    }
  }
}
</script>

<style scoped lang="scss">
.Toolbox {
  height: 50px;
  background-color: lightblue;
  display: flex;
}
</style>
