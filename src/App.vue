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
            :class="elevators[index].arrived ? 'elevator arrived' : 'elevator'" :style="{
              marginBottom: elevators[index].marginBottom,
              transitionProperty: elevators[index].marginBottom,
              transitionDuration: `${elevators[index].distance}s`,
              height: `${height}px`
            }">
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
      lockedFloors: [],

      queueIsNotEmpty: false,
      prikol: false,
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
        targetFloor: 1,
        elevatorState: 'idle',
        localQueue: [],
      }));
    },

    callElevator(floor) {
      if (this.lockedFloors.includes(floor)) {
        return;
      }
      let nearestElevator = -1;
      let nearestDistance = Number.MAX_VALUE;
      for (let i = 0; i < this.elevatorsAmount; i++) {
        let distance = Math.abs(this.elevators[i].elevatorPrevPosition - (floor - 1));
        if (this.elevators[i].elevatorState === 'idle' && distance < nearestDistance) {
          nearestElevator = i;
          nearestDistance = distance;
        }
      }
      this.queue.includes(floor) ? null : this.queue.push(floor);
      this.activeButtons.includes(floor) ? null : this.activeButtons.push(floor);
      
      if (nearestElevator >= 0) {
        if (this.queue.length > 0) {
          this.moveElevator(nearestElevator, floor);
        }
      }
    },

    moveElevator(i, floor) {
      if (!this.queue.length) {
        return;
      }
      this.lockedFloors.splice(this.lockedFloors.indexOf(this.elevators[i].targetFloor), 1);
      this.lockedFloors.push(floor);

      this.elevators[i].targetFloor = this.queue.shift();
      this.elevators[i].elevatorState = parseInt(this.elevators[i].marginBottom) > (this.elevators[i].targetFloor - 1) * this.height ? 'moving down' : 'moving up';
      this.elevators[i].marginBottom = (this.elevators[i].targetFloor - 1) * this.height + 'px';
      this.elevators[i].distance = Math.abs((this.elevators[i].targetFloor - 1) - this.elevators[i].elevatorPrevPosition);
      this.elevators[i].elevatorPrevPosition = (this.elevators[i].targetFloor - 1);
      this.elevators[i].displayText = `${this.elevators[i].targetFloor}`;

      setTimeout(() => {
        this.elevators[i].arrived = true;
        const index = this.activeButtons.indexOf(this.elevators[i].targetFloor);
        if (index !== -1) {
          this.activeButtons.splice(index, 1);
        }
        setTimeout(() => {
          this.elevators[i].elevatorState = 'idle';
          this.elevators[i].arrived = false;
          this.queue.length > 0 ? this.callElevator(this.queue[0]) : null; 
        }, 3000);
      }, this.elevators[i].distance * 1000);
    },

    saveState() {
      const state = {
        marginBottom: [],
        displayText: [],
        targetFloor: [],
        elevatorPrevPosition: [],
      };
      for (let i = 0; i < this.elevatorsAmount; i++) {
        state.marginBottom.push(this.elevators[i].marginBottom);
        state.displayText.push(this.elevators[i].displayText);
        state.targetFloor.push(this.elevators[i].targetFloor);
        state.elevatorPrevPosition.push(this.elevators[i].elevatorPrevPosition);
      }
      sessionStorage.setItem('liftSystemState', JSON.stringify(state));
    },

    arrowState(index) {
        switch (true) {
          case (this.elevators[index].elevatorState === "moving up"):
            return 'arrow-up'
          case (this.elevators[index].elevatorState === "moving down"):
            return 'arrow-down'
          default:
            return 'arrow'
        }
    },

    fillFloorsArray() {
      this.floors = Array.from({ length: this.floorsAmount }, (_, i) => this.floorsAmount - i);
    },

    fillLockedFloors() {
      for (let i = 0; i < this.elevatorsAmount; i++) {
        this.lockedFloors.push(this.elevators[i].targetFloor)
      }
    },

  },

  mounted() {
    this.fillFloorsArray();
    this.fillElevatorsArray();
    const savedState = JSON.parse(sessionStorage.getItem('liftSystemState'));
    if (savedState) {
      for (let i = 0; i < this.elevatorsAmount; i++) {
        this.elevators[i].marginBottom = savedState.marginBottom[i];
        this.elevators[i].elevatorPrevPosition = savedState.elevatorPrevPosition[i];
        this.elevators[i].displayText = savedState.displayText[i];
        this.elevators[i].targetFloor = savedState.targetFloor[i];
      }
    }
    this.fillLockedFloors();
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