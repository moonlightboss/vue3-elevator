<template>
  <div class="elevator" :style="elevatorStyle"></div>
</template>

<script>
export default {
  props: {
    currentFloor: {
      type: Number,
      required: true
    },
    elevatorState: {
      type: String,
      required: true
    },
    targetFloor: {
      type: Number
    },
    isMoving: {
      type: Boolean,
      default: false
    }
  },
  computed: {
    elevatorStyle() {
      return {
        top: `${100 * (6 - this.currentFloor - 1)}px` // Вычисляем позицию лифта, что бы движение было сверху вниз.
      };
    }
  }
};
</script>

<style scoped>
.elevator {
  position: absolute;
  height: 100px;
  width: 100px;
  background-color: blueviolet;
  transition: top 0.5s;
}
</style>
