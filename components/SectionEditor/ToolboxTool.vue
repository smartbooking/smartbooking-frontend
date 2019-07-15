<template>
  <div class="Tool" @click="onSelect" :class="{ active: isActive }">
    <i class="material-icons">
      {{ icon }}
    </i>
  </div>
</template>

<script>
export default {
  props: {
    icon: {
      type: String,
      required: true
    },
    tool: {
      type: String,
      required: true
    }
  },
  data() {
    return {
      isActive: false
    }
  },
  methods: {
    onSelect() {
      if (this.isActive) return

      this.$emit('select', this)
    },
    makeActive() {
      this.isActive = true
    },
    makeInactive() {
      this.isActive = false
    }
  }
}
</script>

<style scoped lang="scss">
.Tool {
  display: flex;
  border-right: 1px solid white;
  cursor: pointer;
  &:hover {
    color: red;
  }
  &.active {
    color: red;
  }
  i {
    font-size: 50px;
  }
}
</style>
