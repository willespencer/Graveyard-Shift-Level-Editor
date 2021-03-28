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
    <div class="toolsAndMapWrapper" v-if="dimensionsSet">
      <tool-bar @update-tile-placing="updateTilePlacing" />
      <level-map
        :height="height"
        :width="width"
        :typeToPlace="typePlacing"
        @tile-changed="updateTiles"
      />
    </div>
    <div v-if="dimensionsSet" class="outputWrapper">
      <span>Level Number:</span>
      <input class="input" v-model="levelNumber" />
      <button @click="writeJSON" :disabled="!areRulesMet">Output File</button>
      <div class="instructions">
        <h3 class="instructionsTitle">
          Instructions For Downloading and Playing the Level
        </h3>
        <ol class="instructionsList">
          <li>
            Set the level number as 1 + the current max level in the codebase.
          </li>
          <li>
            After downloading, add the level to /core/assets/levels/
          </li>
          <li>
            Set the maxLevels variable in GameplayController to this level
            number.
          </li>
          <li>
            Add an import for this new JSON file in /core/assets/assets.json
          </li>
          <li>
            After all of this, the level should be playable!
          </li>
        </ol>
      </div>
    </div>
  </div>
</template>

<script>
import LevelMap from "@/components/LevelMap.vue";
import ToolBar from "@/components/ToolBar.vue";

// UPDATE THE VERSION NUMBER WHEN THE JSON CHANGES
// TODO - convert old versions to new versions somehow
const versionNumber = "1.0";

export default {
  components: { LevelMap, ToolBar },
  data() {
    return {
      width: 0,
      height: 0,
      rows: "",
      cols: "",
      levelNumber: 0,
      dimensionsSet: false,
      typePlacing: "floor",
      tiles: [],
    };
  },
  computed: {
    // return true if the current rules of the map are met
    // rules currently require there to be exactly one player and one goal tile
    areRulesMet() {
      if (this.tiles.length > 0) {
        let players = this.findObjects("player");
        let goals = this.findObjects("goal");
        return players.length === 1 && goals.length === 1;
      }
      return false;
    },
  },
  methods: {
    // update the board dimensions on button click
    updateDimensions() {
      this.width = Number(this.cols);
      this.height = Number(this.rows);
      this.dimensionsSet = true;
    },
    // when the tiles are updated in LevelMap, update them in Dashboard for JSON purposes
    updateTiles(tiles) {
      this.tiles = tiles;
      console.log(this.tiles);
    },
    // update the tile being placed when a button is clicked on in ToolBar
    updateTilePlacing(type) {
      this.typePlacing = type;
    },
    // create the json based on the map and call the output method
    writeJSON() {
      let json = {};
      json["version"] = versionNumber;

      let mutantSpawns = this.findObjects("mutant");
      let brickSpawns = this.findObjects("brick");
      let bombSpawns = this.findObjects("bomb");
      let keySpawns = this.findObjects("key");
      let itemSpawns = this.generateItemSpawns(
        brickSpawns,
        bombSpawns,
        keySpawns
      );

      // set metadata
      let metadata = {
        width: this.width,
        height: this.height,
        "player-spawn": this.findObjects("player")[0],
        "mutant-count": mutantSpawns.length,
        "mutant-spawns": mutantSpawns,
        "item-count": itemSpawns.length,
        "item-spawns": itemSpawns,
      };
      json["metadata"] = metadata;

      // set layout
      let layout = [];
      for (let r = this.height - 1; r >= 0; r--) {
        for (let c = 0; c < this.width; c++) {
          if (this.tiles[r][c] === "wall") {
            layout.push(1);
          } else if (this.tiles[r][c] === "glass") {
            layout.push(2);
          } else if (this.tiles[r][c] === "door") {
            layout.push(5);
          } else if (this.tiles[r][c] === "goal") {
            layout.push(4);
          } else {
            layout.push(0);
          }
        }
      }
      json["layout"] = layout;
      console.log(json);

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
    // returns a 2d array representing all positions of objectType
    // used to find players, mutants, bombs, bricks, keys, etc.
    // TODO - added transforms to here and other methods to match json, maybe json file should be changed
    findObjects(objectType) {
      let objectList = [];
      for (let r = 0; r < this.height; r++) {
        for (let c = 0; c < this.width; c++) {
          if (this.tiles[r][c] === objectType) {
            objectList.push([c, this.height - r - 1]);
          }
        }
      }
      return objectList;
    },
    // returns the json representation of all the item types and their locations
    generateItemSpawns(bricks, bombs, keys) {
      let itemSpawns = [];
      bricks.forEach((b) => {
        let item = {};
        item.type = "BRICK";
        item.position = b;
        itemSpawns.push(item);
      });

      bombs.forEach((b) => {
        let item = {};
        item.type = "BOMB";
        item.position = b;
        itemSpawns.push(item);
      });

      keys.forEach((k) => {
        let item = {};
        item.type = "KEY";
        item.position = k;
        itemSpawns.push(item);
      });

      return itemSpawns;
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

.toolsAndMapWrapper {
  display: flex;
  justify-content: center;
}

.outputWrapper {
  margin-top: 1rem;
}

.instructions {
  display: flex;
  flex-direction: column;
  justify-content: left;
  text-align: left;
}

.instructionsTitle {
  margin-bottom: 0;
  margin-top: 2rem;
}
</style>
