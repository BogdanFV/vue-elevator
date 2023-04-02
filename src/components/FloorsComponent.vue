<template>
    <div class="floors">
        <div v-for="floor in floors" class="floor" :key="floor">
            <button class="call-button" :class="{ 'active': activeButtons.includes(floor) }"
                :style="{ 'background-color': activeButtons.includes(floor) ? 'red' : 'white' }"
                @click="callElevator(floor)">
                {{ floor }}
            </button>
        </div>
    </div>
</template>
<script>
export default {
    props: {
        floors: Array,
        activeButtons: Array,
    },
    methods: {
        callElevator(floor) {
            this.$emit("call-elevator", floor);
        },
    },
};
</script>
<style scoped>
.active {
  background-color: crimson;
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
  border-right: 1px solid black;
}
.call-button:not(.active):hover {
  cursor: pointer;
  height: 42%;
  width: 32%;
  filter: drop-shadow(0 0 0.1rem crimson);
}
.call-button {
  height: 40%;
  width: 30%;
}
</style>