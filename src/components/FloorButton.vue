<template>
  <button
      :class="{ active: isActive, processing: isProcessing, queued: isQueued }"
      @click="callElevator(floor)"
  >
    {{ floor }}
  </button>
</template>

<script>
export default {
  props: {
    floor: {
      type: Number,
      required: true
    },
    isActive: {
      type: Boolean,
      default: false
    },
    isProcessing: {
      type: Boolean,
      default: false
    },
    isQueued: {
      type: Boolean,
      default: false
    }
  },
  methods: {
    callElevator(floor) {
      this.$emit('callElevator', floor);
    }
  }
};
</script>

<style scoped>
button {
  margin: 5px;
  padding: 10px;
  background-color: #f0f0f0;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

button.active {
  background-color: #00ff00;
}

button.processing {
  background-color: #ffff00;
}

button.queued {
  background-color: #ff00ff;
}
</style>
