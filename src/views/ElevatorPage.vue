<template>
  <div>
    <div>
      <p>Текущий этаж: {{ currentFloor }}</p>
      <p>Состояние лифта: {{ elevatorState }}</p>
      <p v-if="elevatorState === 'Движение'">Целевой этаж: {{ targetFloor }}</p>
    </div>

    <div class="elevator-container">
      <div class="elevator" :style="elevatorStyle"></div>
    </div>
    <div class="floor-buttons">
      <button :class="{ active: isActive, processing: floor === targetFloor, queued: queue.includes(floor) }"
              v-for="floor in numberOfFloors" :key="floor" @click="callElevator(floor)">
        Вызов на этаж {{ floor }}
      </button>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    numberOfFloors: {
      type: Number,
      default: 5
    },
    numberOfShafts: {
      type: Number,
      default: 1
    }
  },
  data() {
    return {
      currentFloor: 1,
      elevatorState: 'Покой',
      queue: [],
      targetFloor: null,
      isMoving: false
    };
  },
  computed: {
    isProcessing() {
      return this.elevatorState === 'Движение' && this.targetFloor === this.currentFloor;
    },
    elevatorStyle() {
      return {
        top: `${100 * (this.currentFloor - 1)}px` // Вычисляем позицию лифта
      };
    }
  },
  methods: {
    callElevator(floor) {
      if (
          this.currentFloor === floor ||
          this.elevatorState === 'Движение' ||
          this.queue.includes(floor)
      ) {
        return;
      }

      if (this.elevatorState === 'Покой') {
        this.moveElevator(floor);
      } else {
        this.queue.push(floor);
      }

      this.targetFloor = floor; // Установка целевого этажа
      this.saveState(); // Сохранение состояния
    },
    moveElevator(floor) {
      this.elevatorState = 'Движение';
      this.isMoving = true; // Установка флага движения

      const distance = Math.abs(this.currentFloor - floor);
      const duration = distance * 1000; // 1 этаж в секунду
      const intervalDuration = 1000; // Интервал обновления позиции лифта

      let elapsedTime = 0;
      const interval = setInterval(() => {
        elapsedTime += intervalDuration;

        if (elapsedTime >= duration) {
          clearInterval(interval);
          this.currentFloor = floor;
          this.elevatorState = 'Отдых';
          this.rest();
          this.saveState(); // Сохранение состояния
        } else {
          const progress = elapsedTime / duration;
          const floorDifference = floor - this.currentFloor;
          this.currentFloor = this.currentFloor + Math.round(floorDifference * progress);
        }
      }, intervalDuration);
    },
    rest() {
      setTimeout(() => {
        this.elevatorState = 'Покой';

        if (this.queue.length > 0) {
          const nextFloor = this.queue.shift();
          this.moveElevator(nextFloor);
        }

        this.saveState(); // Сохранение состояния
      }, 3000);
    },
    saveState() {
      const state = {
        currentFloor: this.currentFloor,
        elevatorState: this.elevatorState,
        queue: this.queue,
        targetFloor: this.targetFloor
      };
      localStorage.setItem('elevatorState', JSON.stringify(state));
    },
    restoreState() {
      const savedState = localStorage.getItem('elevatorState');
      if (savedState) {
        const state = JSON.parse(savedState);
        this.currentFloor = state.currentFloor;
        this.elevatorState = state.elevatorState;
        this.queue = state.queue;
        this.targetFloor = state.targetFloor;
      }
    }
  },
  mounted() {
    this.restoreState(); // Восстановление состояния при загрузке страницы
  }
};
</script>

<style>
.elevator-container {
  position: relative;
  height: 500px;
  width: 100px;
  margin-bottom: 20px;
}

.elevator {
  position: absolute;
  top: 0;
  left: 0;
  height: 100px;
  width: 100px;
  background-color: blue;
  transition: top 1s linear; /* Добавляем анимацию движения */
}

.floor-buttons button {
  margin-right: 10px;
}

.processing {
  background-color: blue;
  color: white;
}
</style>
