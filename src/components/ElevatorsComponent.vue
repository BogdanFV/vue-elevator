<template>
    <div class="elevators">
        <div v-for="(elevator, index) in elevators" :key="elevator" :state="elevators[index].elevatorState"
            :class="elevators[index].arrived ? 'elevator arrived' : 'elevator'" :style="{
                marginBottom: elevators[index].marginBottom,
                transitionProperty: elevators[index].marginBottom,
                transitionDuration: `${elevators[index].distance}s`,
                height: `${height}px`
            }">
            <div class="elevator-info">
                <img alt="arrow" src="../../src/assets/arrow-up.svg" :class="arrowState(index)">
                <div class="display">{{ elevators[index].displayText }}</div>
            </div>
        </div>
    </div>
</template>
<script>
export default {
    props: {
        elevators: Array,
        height: Number,
    },
    methods: {
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

    },
};
</script>
<style scoped>

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

.display {
  color: wheat;
  font-size: 30px;
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

.elevator-info {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: center;
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