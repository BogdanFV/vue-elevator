<template>
  <div class="app-cover">
    <div class="lift-system">
      <div class="floors">
        <div v-for="floor in floors" class="floor" :key="floor">
          <button class="call-button" :class="{ 'active': activeButtons.includes(floor) }"
            :style="{ 'background-color': activeButtons.includes(floor) ? 'red' : 'white' }" @click="callElevator(floor)">
            {{ floor }}
          </button>
        </div>
      </div>
      <div class="shaft"  :style="{ height: `${height * floorsAmount}px` }">
        <div :state="elevatorState" :class="arrived ? 'elevator arrived' : 'elevator'" :style="{
          marginBottom,
          transitionProperty: marginBottom,
          transitionDuration: `${distance}s`,
        }">

          <div class="elevator-container">
            <img alt="arrow" src="../src/assets/arrow-up.svg" :class="arrowState()">
            <div class="display">{{ displayText || 1 }}</div>
          </div>``
        </div>
      </div>
    </div>
  </div>
</template>


<script>
export default {
  data() {
    return {
      activeButtons: [],
      arrived: false,
      displayText: '',
      distance: null,
      elevatorPosition: 0,
      direction: 'idle',

      floorsAmount: 7,
      floors: [],
      height: 100,
      isMoving: false,

      marginBottom: sessionStorage.getItem('marginBottom') || 0,
      targetFloor: null,
      queue: [],
    };
  },
  methods: {

    fillFloorsArray() {
      this.floors = Array.from({ length: this.floorsAmount }, (_, i) => this.floorsAmount - i);
    },

    callElevator(floor) {
      if (this.queue.includes(floor) || this.elevatorPosition === (floor - 1)) {
        return;
      }

      this.activeButtons.push(floor);
      this.queue.push(floor);
      if (!this.isMoving) {
        this.moveElevator();
      }
    },

    moveElevator(floor) {
      if (!this.queue.length) {
        this.isMoving = false;
        this.direction = 'idle';
        return;
      }
      this.elevatorState = "moving down"
      this.movingUp = this.elevatorPosition > this.targetFloor - 1;
      this.currentFloor = floor;
      this.isMoving = true;
      this.targetFloor = this.queue.shift();


      if (this.marginBottom > (this.targetFloor - 1) * this.height + 'px') {
        this.direction = 'down';
      } else {
        this.direction = 'up';
      }

      this.marginBottom = (this.targetFloor - 1) * this.height + 'px';

      this.distance = Math.abs((this.targetFloor - 1) - this.elevatorPosition);

      this.elevatorPosition = (this.targetFloor - 1);
      this.displayText = `${this.targetFloor}`;
      sessionStorage.setItem('marginBottom', this.marginBottom);


      setTimeout(() => {
        this.arrived = true;
        const index = this.activeButtons.indexOf(this.targetFloor);
        if (index !== -1) {
          this.activeButtons.splice(index, 1);
        }
        setTimeout(() => {
          this.arrived = false;
          this.moveElevator(floor);
        }, 3000);
      }, this.distance * 1000)
    },

    saveState() {
      const state = {
        elevatorPosition: this.elevatorPosition,
        queue: this.queue,
        displayText: this.displayText,
        targetFloor: this.targetFloor
      };
      sessionStorage.setItem('liftSystemState', JSON.stringify(state));
    },

    handleQueue() {
      if (!this.isMoving) {
        this.moveElevator();
      }
    },

    arrowState() {
      if (this.isMoving) {
        if (this.direction === "up") {
          return 'arrow-up'
        } else if (this.direction === "down") {
          return 'arrow-down'
        }
      } else {
        return 'arrow'
      }
    }

  },

  mounted() {
    this.fillFloorsArray();
    const savedState = JSON.parse(sessionStorage.getItem('liftSystemState'));
    if (savedState) {
      this.elevatorPosition = savedState.elevatorPosition;
      this.queue = savedState.queue;
      this.displayText = savedState.displayText;
      this.targetFloor = savedState.targetFloor;
    }
    window.addEventListener('beforeunload', this.saveState);
  },

  beforeUnmount() {
    this.saveState();
    window.removeEventListener('beforeunload', this.saveState);
  },
}
</script>
<style>
body {
  margin: 0;
}
.app-cover{
  display: flex;
  align-items: center;
  justify-content: center;
  height: 100vh;
  background-color: #a1f2a4;
}

.arrow {
  display: none;
  height: 30px;
  width: 30px;
}

.arrow-up {
  height: 30px;
  width: 30px;
}

.arrow-down {
  height: 30px;
  width: 30px;
  transform: rotate(180deg);
}

.lift-system {
  display: flex;
  justify-content: center;
  align-items: flex-end;
  flex-direction: row;
}

.floors {
  display: flex;
  background: grey;
  flex-direction: column;
  justify-content: flex-end;
  align-items: center;
}

.floor {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100px;
  width: 16vh;
}

.elevator {
  position: absolute;
  bottom: 0;
  height: 100px;
  width: 16vh;
}

.shaft {
  width: 16vh;
  position: relative;
  background: orange;
}

.elevator {
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: rgb(23, 23, 29);
}

.elevator-container {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: center;
}

.display {
  color: wheat;
  font-size: 30px;
}

.call-button:hover {
  cursor: pointer;
}

.call-button {
  height: 40%;
  width: 30%;
}

.active {
  background-color: red;
}

@keyframes blink {
  0% {
    opacity: 1;
  }

  50% {
    opacity: 0.5;
  }

  100% {
    opacity: 1;
  }
}

.arrived {
  animation: blink 0.5s ease-in-out 6;
}
</style>