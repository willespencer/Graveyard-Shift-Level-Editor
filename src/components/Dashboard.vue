<template>
  <div class="dashboard">
    <h1 class="title">Level Editor</h1>
    <h4 class="version">Version {{ version }}</h4>
    <div class="topWrapper" v-if="!displayMap">
      <div class="optionWrapper">
        <h3>New Level</h3>
        <span>What size should the level be (in tiles, width x height)?</span>
        <div class="sizeInputs">
          <input class="input" v-model="cols" /> x
          <input class="input" v-model="rows" />
        </div>
        <button class="button" @click="updateDimensions">
          Create Map
        </button>
      </div>
      <div class="optionWrapper">
        <h3>Existing Level</h3>
        <input type="file" ref="myFiles" class="fileInput" />
        <button class="button" @click="loadMap">Load Map</button>
      </div>
    </div>
    <div class="toolsAndMapWrapper" v-if="displayMap">
      <tool-bar @update-tile-placing="updateTilePlacing" />
      <level-map
        :height="height"
        :width="width"
        :typeToPlace="typePlacing"
        :inputTiles="inputTiles"
        @tile-changed="updateTiles"
      />
    </div>
    <div v-if="displayMap" class="outputWrapper">
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
const stringify = require("json-stringify-pretty-compact");

// UPDATE THE VERSION NUMBER WHEN THE JSON CHANGES
// TODO - convert old versions to new versions somehow
const versionNumber = "1.1";

export default {
  components: { LevelMap, ToolBar },
  data() {
    return {
      width: 0,
      height: 0,
      rows: "",
      cols: "",
      levelNumber: 0,
      displayMap: false,
      typePlacing: "floor",
      tiles: [],
      inputTiles: [],
      version: versionNumber,
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
      this.displayMap = true;
    },
    // when the tiles are updated in LevelMap, update them in Dashboard for JSON purposes
    updateTiles(tiles) {
      this.tiles = tiles;
    },
    // update the tile being placed when a button is clicked on in ToolBar
    updateTilePlacing(type) {
      this.typePlacing = type;
    },
    // load in an existing map
    // code adapted from: https://stackoverflow.com/questions/59155812/vue-upload-local-json-file
    loadMap() {
      const files = this.$refs.myFiles.files;
      if (files.length <= 0) {
        return false;
      }

      const fr = new FileReader();

      fr.onload = (e) => {
        const result = JSON.parse(e.target.result);
        this.convertJSONToMap(result);
      };
      fr.readAsText(files.item(0));
    },
    // converts a loaded in JSON to the parameters needed to display it
    convertJSONToMap(json) {
      // set width and height
      this.width = json.metadata.width;
      this.height = json.metadata.height;

      // copy tile layout in
      let jsonLayout = json.layout;
      let tiles = new Array(this.height)
        .fill(0)
        .map(() => new Array(this.width).fill(0));
      for (let i = 0; i < jsonLayout.length; i++) {
        let r = this.height - 1 - Math.floor(i / this.width);
        let c = i % this.width;
        tiles[r][c] = this.convertNumberToTile(jsonLayout[i]);
      }

      // copy item spawns in
      let itemSpawns = json.metadata["item-spawns"];
      for (let i = 0; i < itemSpawns.length; i++) {
        let position = itemSpawns[i].position;
        let type = itemSpawns[i].type.toLowerCase();
        tiles[this.height - position[1] - 1][position[0]] = type;
      }

      // copy player mutants in
      let mutantSpawns = json.metadata["mutant-spawns"];
      for (let i = 0; i < mutantSpawns.length; i++) {
        let position = mutantSpawns[i].position;
        let type = mutantSpawns[i].type.toLowerCase();
        tiles[this.height - position[1] - 1][position[0]] = type;
      }

      let playerSpawn = json.metadata["player-spawn"];
      tiles[this.height - playerSpawn[1] - 1][playerSpawn[0]] = "player";

      // set input tiles and display the map
      this.inputTiles = tiles;
      this.displayMap = true;
    },
    // converts the number representation of a tile in the game to the label in the editor
    convertNumberToTile(num) {
      if (num == 1) {
        return "wall";
      } else if (num == 2) {
        return "glass";
      } else if (num == 5) {
        return "door";
      } else if (num == 4) {
        return "goal";
      } else if (num == 6) {
        return "barrel";
      } else if (num == 7) {
        return "grate";
      } else {
        return "floor";
      }
    },
    // converts the tile name in the editor to the number representation of a tile in the game
    convertTileToNumber(tile) {
      if (tile == "wall") {
        return 1;
      } else if (tile == "glass") {
        return 2;
      } else if (tile == "door") {
        return 5;
      } else if (tile == "goal") {
        return 4;
      } else if (tile == "barrel") {
        return 6;
      } else if (tile == "grate") {
        return 7;
      } else {
        return 0;
      }
    },
    // create the json based on the map and call the output method
    writeJSON() {
      let json = {};
      json["version"] = this.version;

      let normalMutantSpawns = this.findObjects("normal");
      let acuteMutantSpawns = this.findObjects("acute");
      let brickSpawns = this.findObjects("brick");
      let bombSpawns = this.findObjects("bomb");
      let keySpawns = this.findObjects("key");
      let itemSpawns = this.generateItemSpawns(
        brickSpawns,
        bombSpawns,
        keySpawns
      );
      let mutantSpawns = this.generateMutantSpawns(
        normalMutantSpawns,
        acuteMutantSpawns
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
          layout.push(this.convertTileToNumber(this.tiles[r][c]));
        }
      }
      json["layout"] = layout;

      this.outputJSONToFile(json);
    },
    // output json to a file with provided level number and automatically download it to the user's computer
    outputJSONToFile(json) {
      // code source: https://stackoverflow.com/questions/48611671/vue-js-write-json-object-to-local-file
      const data = stringify(json, null, 2);
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
    // generate the json representation of mutant spawns based on their type
    // TODO - code is repetitive with generateItemSpawns, combine
    generateMutantSpawns(normal, acute) {
      let mutantSpawns = [];
      normal.forEach((m) => {
        let mutant = {};
        mutant.type = "NORMAL";
        mutant.position = m;
        mutantSpawns.push(mutant);
      });

      acute.forEach((m) => {
        let mutant = {};
        mutant.type = "ACUTE";
        mutant.position = m;
        mutantSpawns.push(mutant);
      });

      return mutantSpawns;
    },
  },
};
</script>

<style scoped>
.title {
  margin-bottom: 0;
}

.version {
  margin-top: 0;
}

.topWrapper {
  display: flex;
  justify-content: space-around;
}
.optionWrapper {
  display: flex;
  align-items: center;
  flex-direction: column;
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
  min-width: 90px;
}

.fileInput {
  border: 1px gray solid;
  padding: 3px;
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
