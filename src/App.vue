<template>
  <div class="lift-system">
    <div class="floors">
      <div v-for="floor in floors" :key="floor" class="floor">
        <!--:disabled="isButtonDisabled(floor)" -->
        <button class="call-button"  @click="callElevator(floor)">{{floor}}</button>
      </div>
    </div>
    <div class="shaft">
      <div class="elevator" :style="{ transform: 'translateY(' + elevatorPosition + 'px)' }">
        <div class="elevator-container">
          <div class="display">{{ displayText }}</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      floors: Array.from({length: 7}, (_, i) => i + 1),
      elevatorPosition: 0,
      isMoving: false,
      targetFloor: null,
      queue: [],
      displayText: '1',
    };
  },
  methods: {
    callElevator(floor) {
      if (this.queue.includes(floor) || this.elevatorPosition === (floor - 1) * 100) {
        return;
      }

      this.queue.push(floor);

      if (!this.isMoving) {
        this.moveElevator();
      }
    },
    moveElevator() {
      if (!this.queue.length) {
        this.isMoving = false;
        // this.displayText = 'Elevator is idle';
        return;
      }
      this.isMoving = true;
      this.targetFloor = this.queue.shift();
      // this.displayText = `Moving ${this.targetFloor > this.currentFloor ? 'up' : 'down'} to floor ${this.targetFloor}`;

      const distance = Math.abs((this.targetFloor - 1) * 100 - this.elevatorPosition);
      const duration = distance * 10;

      setTimeout(() => {
        this.elevatorPosition = (this.targetFloor - 1) * 100;
        this.displayText = `${this.targetFloor}`;
        // this.displayText = `Arrived at floor ${this.targetFloor}`;

        setTimeout(() => {
          // this.displayText = 'Elevator is idle';
          this.moveElevator();
        }, 3000);
      }, duration);
    },
    selectFloor(floor) {
      if (this.isMoving || this.queue.includes(floor)) {
        return;
      }

      this.queue.push(floor);

      if (this.targetFloor === null) {
        this.moveElevator();
      }
    },
    isButtonDisabled(floor) {
      return this.isMoving || this.elevatorPosition === (floor - 1) * 100;
    },
    isButtonActive(floor) {
      return this.targetFloor === floor;
    },
  },
  mounted() {
    const savedState = JSON.parse(localStorage.getItem('liftSystemState'));
    if (savedState) {
      this.elevatorPosition = savedState.elevatorPosition;
      this.queue = savedState.queue;
      this.displayText = savedState.displayText;
      this.targetFloor = savedState.targetFloor;
    }
  },
  created() {
    const savedState = JSON.parse(localStorage.getItem('liftSystemState'));
    if (savedState) {
      this.currentFloor = savedState.currentFloor;
      this.queue = savedState.queue;
    }
  },
  beforeUnmount() {
    const stateToSave = {
      currentFloor: this.currentFloor,
      queue: this.queue
    };
    localStorage.setItem('liftSystemState', JSON.stringify(stateToSave));
  },
  // watch: {
  //   currentFloor: function (newFloor, oldFloor) {
  //     console.log("we are here");
  //     if (newFloor !== oldFloor) {
  //       this.isMoving = true;
  //       setTimeout(() => {
  //         this.isMoving = false;
  //         if (this.queue.length > 0) {
  //           this.handleQueue();
  //         }
  //       }, 3000);
  //     }
  //   },
  //   queue: function (newQueue, oldQueue) {
  //     if (newQueue.length > oldQueue.length) {
  //       this.handleQueue();
  //     }
  //   }
  // },
}

</script>
<style>
body {
  margin: 0;
}
div {
  /* border: 1px solid black; */
}
.lift-system{
  display: flex;
  flex-direction: row;
  background-color: #a1f2a4;
}
.floor{
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100px;
  width: 16vh;
}
.elevator{
  height: 100px;
  width: 16vh;
}
.shaft{
  background: red;
}
.elevator{
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: blue;
}
.elevator-container{
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: center;
}
.display{
  color: wheat;
  font-size: 30px;
}
.call-button{
  height: 40%;
  width: 30%;
}
</style>