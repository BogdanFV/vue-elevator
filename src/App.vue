Богдан, [03.04.2023 1:45]
<template>
  <div class="app-cover">
    <div class="lift-system">
      <FloorsComponent :floors="floors" :active-buttons="activeButtons" @call-elevator="callElevator" />
      <div class="shaft" :style="{ height: `${ height * floorsAmount }px` }">
        <ElevatorsComponent :elevators="elevators" :height="height" />
      </div>
    </div>
    <div class="ground"></div>
  </div>
</template>
<script>
import FloorsComponent from "@/components/FloorsComponent";
import ElevatorsComponent from "@/components/ElevatorsComponent";
import config from '../config.json';

export default {
  components: {
    FloorsComponent,
    ElevatorsComponent,
  },
  data() {
    return {
      floorsAmount: config.floorsAmount,
      height: 100,
      elevatorsAmount: 3,

      queue: [],
      activeButtons: [],
      floors: [],
      elevators: [],
      lockedFloors: [],
    };
  },
  methods: {
    fillElevatorsArray() {
      this.elevators = Array.from({ length: this.elevatorsAmount }, () => ({
        arrived: false,
        displayText: 1,
        distance: null,
        elevatorPrevFloor: 1,
        marginBottom: 0,
        targetFloor: 1,
        elevatorState: 'idle',
      }));
    },

    callElevator(floor) {
      if (this.lockedFloors.includes(floor)) {
        return;
      }
      let nearestElevator = -1;
      let nearestDistance = Number.MAX_VALUE;
      for (let i = 0; i < this.elevatorsAmount; i++) {
        let distance = Math.abs(this.elevators[i].targetFloor - floor);
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

      this.elevators[i].targetFloor = floor;
      this.queue.shift();
      this.elevators[i].elevatorState = parseInt(this.elevators[i].marginBottom) > (this.elevators[i].targetFloor - 1) * this.height ? 'moving down' : 'moving up';
      this.elevators[i].marginBottom = `${(this.elevators[i].targetFloor - 1) * this.height}px`;
      this.elevators[i].distance = Math.abs(this.elevators[i].targetFloor - this.elevators[i].elevatorPrevFloor);
      this.elevators[i].displayText = `${this.elevators[i].targetFloor}`;

      setTimeout(() => {
        this.elevators[i].arrived = true;
        const index = this.activeButtons.indexOf(this.elevators[i].targetFloor);
        if (index !== -1) {
          this.activeButtons.splice(index, 1);
        }
        this.elevators[i].elevatorPrevFloor = this.elevators[i].targetFloor;
        setTimeout(() => {
          this.elevators[i].elevatorState = 'idle';
          this.elevators[i].arrived = false;
          this.queue.length > 0 ? this.callElevator(this.queue[0]) : null;
        }, 3000);
      }, this.elevators[i].distance * 1000);
    },
    saveState() {
      const state = {
        queue: this.queue,
        activeButtons: this.activeButtons,
        arrived: [],
        distance: [],
        displayText: [],
        targetFloor: [],
        marginBottom: [],
        elevatorState: [],
        elevatorPrevFloor: [],
      };
      for (let i = 0; i < this.elevatorsAmount; i++) {
        state.displayText.push(this.elevators[i].targetFloor);
        state.marginBottom.push(this.elevators[i].marginBottom);
        state.targetFloor.push(this.elevators[i].targetFloor);
        state.elevatorState.push('idle');
        state.elevatorPrevFloor.push(this.elevators[i].elevatorPrevFloor);
      }
      sessionStorage.setItem('liftSystemState', JSON.stringify(state));
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
        this.elevators[i].arrived = savedState.arrived[i];
        this.elevators[i].displayText = savedState.displayText[i];
        this.elevators[i].targetFloor = savedState.targetFloor[i];
        this.elevators[i].marginBottom = savedState.marginBottom[i];
        this.elevators[i].elevatorState = savedState.elevatorState[i];
        this.elevators[i].elevatorPrevFloor = savedState.elevatorPrevFloor[i];
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
  justify-content: flex-end;
  flex-direction: column;
  height: 100vh;
  background-color: #1b1b1b;
}

.lift-system {
  display: flex;
  justify-content: center;
  align-items: flex-end;
  flex-direction: row;
  border: 1px solid black;
}

.shaft {
  background: orange;
}

.ground {
  width: 100vw;
  height: 10vh;
  background: rgb(63, 63, 63);
}</style>