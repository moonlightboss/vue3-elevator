<template>
  <div>
    <div>
      <p>Текущий этаж: {{ currentFloor }}</p>
      <p>Состояние лифта: {{ elevatorState }}</p>
      <div class="target-floor">
        <p>Целевой этаж: {{ elevatorState === 'Движение' ? targetFloor : '' }}</p>
      </div>
    </div>
    <div class="floor-buttons">
      <floor-button
          v-for="floor in numberOfFloors"
          :key="floor"
          :floor="floor"
          :isActive="currentFloor === floor"
          :isProcessing="floor === targetFloor"
          :isQueued="queue.includes(floor)"
          @callElevator="callElevator"
      ></floor-button>
    </div>
    <div class="elevator-container">
      <elevator-movement
          :currentFloor="currentFloor"
          :elevatorState="elevatorState"
          :targetFloor="targetFloor"
          :isMoving="isMoving"
          @moveElevator="moveElevator"
          @rest="rest"
      ></elevator-movement>
    </div>
  </div>
</template>

<script>
import FloorButton from '../components/FloorButton.vue';
import ElevatorMovement from '../components/ElevatorMovement.vue';

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
      const intervalDuration = 999; // Интервал обновления позиции лифта, 999 что бы проверяло быстрее чем происходит движение

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
  },
  components: {
    FloorButton,
    ElevatorMovement
  }
};
</script>

<style scoped>
.floor-buttons {
  display: flex;
  justify-content: center;
  margin-bottom: 20px;
}

.elevator-container {
  position: relative;
  height: 500px;
  width: 100px;
  margin: 0 auto;
  background-color: #f0f0f0;
}
.elevator {
  position: absolute;
  height: 100px;
  width: 100px;
  background-color: blue;
  transition: top 0.5s;
}
</style>
