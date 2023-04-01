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
      <div class="shaft" :style="{ height: `${height * floorsAmount}px` }">
        <div class="elevators">
          <div v-for="(elevator, index)  in elevators" :key="elevator" :state="elevators[index].elevatorState"
            :class="elevators[index].arrived ? 'elevator arrived' : 'elevator'" 
            :style="{
              marginBottom: elevators[index].marginBottom,
              transitionProperty: elevators[index].marginBottom,
              transitionDuration: `${elevators[index].distance}s`,
              height: `${height}px`
            }"
          >
          <div class="elevator-info">
            <img alt="arrow" src="../src/assets/arrow-up.svg" :class="arrowState(index)">
            <div class="display">{{ elevators[index].displayText }}</div>
          </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>


<script>
export default {
  data() {
    return {
      floorsAmount: 7,
      height: 100,
      elevatorsAmount: 3,

      queue: [],
      activeButtons: [],

      floors: [],
      elevators: [],
      queueIsNotEmpty: false,

      targetFloor: null,
    };
  },
  methods: {
    fillElevatorsArray() {
      this.elevators = Array.from({ length: this.elevatorsAmount }, () => ({
        arrived: false,
        displayText: 1,
        distance: null,
        elevatorPrevPosition: 0,
        marginBottom: sessionStorage.getItem('marginBottom') || 0,
        targetFloor: null,
        elevatorState: 'idle'
      }));
      console.log(this.elevators);
    },

    callElevator(floor) {
      if (this.queue.includes(floor) || this.elevatorPrevPosition === (floor - 1)) {
        return;
      }
      this.activeButtons.push(floor);
      this.queue.push(floor);
      if (!this.queueIsNotEmpty) {
        this.moveElevator();
      }
    },

    moveElevator() {
      if (!this.queue.length) {
        this.queueIsNotEmpty = false;
        return;
      }
      this.queueIsNotEmpty = true;
      this.targetFloor = this.queue.shift();

      this.elevators[0].elevatorState = parseInt(this.elevators[0].marginBottom) > (this.targetFloor - 1) * this.height ? 'moving down' : 'moving up';
      this.elevators[0].marginBottom = (this.targetFloor - 1) * this.height + 'px';
      this.elevators[0].distance = Math.abs((this.targetFloor - 1) - this.elevators[0].elevatorPrevPosition);
      this.elevators[0].elevatorPrevPosition = (this.targetFloor - 1);
      this.elevators[0].displayText = `${this.targetFloor}`;

      setTimeout(() => {
        this.elevators[0].arrived = true;
        const index = this.activeButtons.indexOf(this.targetFloor);
        if (index !== -1) {
          this.activeButtons.splice(index, 1);
        }
        setTimeout(() => {
          this.elevators[0].arrived = false;
          this.moveElevator();
        }, 3000);
      }, this.elevators[0].distance * 1000);
    },

    saveState() {
      const state = {
        marginBottom: this.elevators[0].marginBottom,
        displayText: this.elevators[0].displayText,
        targetFloor: this.elevators[0].targetFloor,
        elevatorPrevPosition: this.elevators[0].elevatorPrevPosition
      };
      sessionStorage.setItem('liftSystemState', JSON.stringify(state));
    },

    handleQueue() {
      if (!this.queueIsNotEmpty) {
        this.moveElevator();
      }
    },

    arrowState(index) {
      if (this.queueIsNotEmpty) {
        switch (true) {
          case (this.elevators[index].elevatorState === "moving up"):
            return 'arrow-up'
          case (this.elevators[index].elevatorState === "moving down"):
            return 'arrow-down'
          default:
            return 'arrow'
        }
      }
      return 'arrow'
    },

    fillFloorsArray() {
      this.floors = Array.from({ length: this.floorsAmount }, (_, i) => this.floorsAmount - i);
    },

  },

  mounted() {
    this.fillElevatorsArray();
    this.fillFloorsArray();
    const savedState = JSON.parse(sessionStorage.getItem('liftSystemState'));
    if (savedState) {
      this.elevators[0].marginBottom = savedState.marginBottom;
      this.elevatorPrevPosition = savedState.elevatorPrevPosition;
      this.elevators[0].displayText = savedState.displayText;
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

.app-cover {
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

.elevators {
  display: flex;
  flex-direction: row;
  align-items: flex-end;
  height: 100%;
}

.elevator {
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: rgb(23, 23, 29);
  bottom: 0;
  width: 16vh;
  border-left: 1px solid white;
  border-right: 1px solid white;
}

.shaft {
  background: orange;
}

.elevator-info {
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