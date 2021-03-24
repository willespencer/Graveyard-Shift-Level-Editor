<template>
  <div class="dashboard">
    <div class="sizeWrapper">
      <span>What size should the level be (in tiles, width x height)?</span>
      <div class="sizeInputs">
        <input class="input" v-model="cols" /> x
        <input class="input" v-model="rows" />
      </div>
      <button class="button" @click="updateDimensions">Create Map</button>
    </div>
    <level-map :height="height" :width="width" />
  </div>
</template>

<script>
import LevelMap from "@/components/LevelMap.vue";

export default {
  components: { LevelMap },
  data() {
    return {
      width: 0,
      height: 0,
      rows: "",
      cols: "",
    };
  },
  methods: {
    updateDimensions() {
      this.width = Number(this.cols);
      this.height = Number(this.rows);
    },
  },
};
</script>

<style scoped>
.sizeWrapper {
  display: flex;
  align-items: center;
  flex-direction: column;
  margin-left: auto;
  margin-right: auto;
}

.sizeInputs {
  margin-top: 0.5rem;
  display: flex;
}

.input {
  max-width: 2rem;
  margin: 0 0.5rem;
}

.button {
  margin-top: 0.5rem;
}
</style>
