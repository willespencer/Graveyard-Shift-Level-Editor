<template>
  <div class="dashboard">
    <h1>Level Editor</h1>
    <div class="sizeWrapper">
      <span>What size should the level be (in tiles, width x height)?</span>
      <div class="sizeInputs">
        <input class="input" v-model="cols" /> x
        <input class="input" v-model="rows" />
      </div>
      <button class="button" @click="updateDimensions">Create Map</button>
    </div>
    <level-map :height="height" :width="width" />
    <div v-if="dimensionsSet" class="outputWrapper">
      <span>Level Number:</span>
      <input class="input" v-model="levelNumber" />
      <button @click="writeJSON">Output File</button>
    </div>
  </div>
</template>

<script>
import LevelMap from "@/components/LevelMap.vue";

// UPDATE THE VERSION NUMBER WHEN THE JSON CHANGES
// TODO - convert old versions to new versions somehow
const versionNumber = "1.0";

export default {
  components: { LevelMap },
  data() {
    return {
      width: 0,
      height: 0,
      rows: "",
      cols: "",
      levelNumber: 0,
      dimensionsSet: false,
    };
  },
  methods: {
    // update the board dimensions on button click
    updateDimensions() {
      this.width = Number(this.cols);
      this.height = Number(this.rows);
      this.dimensionsSet = true;
    },
    // create the json based on the map and call the output method
    writeJSON() {
      // TODO - remove hardcoded values
      let json = {};
      json["version"] = versionNumber;

      // set metadata
      let metadata = {
        width: this.width,
        height: this.height,
        "player-spawn": [1, 1],
        "mutant-count": 1,
        "mutant-spawns": [[2, 2]],
        "item-count": 1,
        "item-spawns": [
          {
            position: [2, 1],
            type: "BRICK",
          },
        ],
      };
      json["metadata"] = metadata;

      // set layout
      let layout = [];
      for (let r = 0; r < this.height; r++) {
        for (let c = 0; c < this.width; c++) {
          layout.push(0);
        }
      }
      json["layout"] = layout;

      this.outputJSONToFile(json);
    },
    // output json to a file with provided level number and automatically download it to the user's computer
    outputJSONToFile(json) {
      // code source: https://stackoverflow.com/questions/48611671/vue-js-write-json-object-to-local-file
      const data = JSON.stringify(json);
      const blob = new Blob([data], { type: "text/plain" });
      const e = document.createEvent("MouseEvents"),
        a = document.createElement("a");
      a.download = "level" + this.levelNumber + ".json";
      a.href = window.URL.createObjectURL(blob);
      a.dataset.downloadurl = ["text/json", a.download, a.href].join(":");
      e.initEvent(
        "click",
        true,
        false,
        window,
        0,
        0,
        0,
        0,
        0,
        false,
        false,
        false,
        false,
        0,
        null
      );
      a.dispatchEvent(e);
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

.outputWrapper {
  margin-top: 1rem;
}
</style>
