<template>
  <div class="dashboard">
    <img class="logo" alt="Logo" src="@/assets/logo_new.png" />
    <h1 class="title">Level Editor</h1>
    <h4 class="version">File Version {{ version }}</h4>
    <div class="topWrapper">
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
        <div class="fileWrapper">
          <input type="file" id="file" ref="myFiles" class="fileInput" />
          <label class="fileLabelButton" for="file">Choose File</label>
          <span v-if="fileName" class="fileLabel">{{ fileName }}</span>
        </div>
        <button class="button" @click="loadMap" :disabled="!fileInputted">
          Load Map
        </button>
      </div>
    </div>
    <div class="toolsAndMapWrapper" v-if="displayMap">
      <tool-bar
        class="toolbar"
        :triggerToEdit="triggerToEdit"
        @update-tile-placing="updateTilePlacing"
        @add-trigger="addTrigger"
      />
      <level-map
        class="map"
        :dimensions="dimensions"
        :typeToPlace="typePlacing"
        :inputTiles="inputTiles"
        :inputObjects="inputObjects"
        :inputPaths="inputPaths"
        :levelLoading="levelLoading"
        :triggerList="triggerList"
        @level-loaded="setLevelLoaded"
        @tile-changed="updateTilesAndObjects"
      />
    </div>
    <finished-triggers
      class="finishedTriggers"
      :triggerList="triggerList"
      v-if="typePlacing === 'trigger'"
      @delete-trigger="deleteTrigger"
      @edit-trigger="editTrigger"
    />
    <div v-if="displayMap" class="outputWrapper">
      <span>Level Number:</span>
      <input class="input" v-model="levelNumber" />
      <button @click="writeJSON" :disabled="!areRulesMet">Output File</button>
      <div class="instructions">
        <h3 class="instructionsTitle">Rules</h3>
        <ol class="instructionsList">
          <li class="instructionsItem">
            There must be exactly 1 player on the level
          </li>
          <li class="instructionsItem">
            There must be at least 1 goal on the level
          </li>
        </ol>
        <span class="instructionsDesc"
          >If these rules are not followed, level cannot be downloaded!</span
        >
        <h3 class="instructionsTitle">Mutant Patrol Paths</h3>
        <ol class="instructionsList">
          <li class="instructionsItem">
            Mutants can have patrol paths by clicking the path button in the
            toolbar and clicking on mutants in the patrolling order.
          </li>
          <li class="instructionsItem">
            Clicking a different button in the toolbar will "complete" the
            current path. This path cannot be added to later.
          </li>
          <li class="instructionsItem">
            To remove a path, replace one of the mutant icons in the patrol path
            with a different tile.
          </li>
          <li class="instructionsItem">
            Note: arrows drawn on the map might not reflect the direction the
            mutant actually travels in the game.
          </li>
        </ol>
        <h3 class="instructionsTitle instructionsTitle--last">Trigger Menu</h3>
        <ol class="instructionsList">
          <li class="instructionsItem">
            Dialogue and Control Triggers can be created and viewed by going to
            the triggers menu.
          </li>
          <li class="instructionsItem">
            Use the icon buttons to edit and delete existing triggers.
          </li>
          <li class="instructionsItem">
            Triggers will show up in their rough locations (if not using
            integers) on the map after being added.
          </li>
          <li class="instructionsItem">
            See the documentation
            <a
              href="https://docs.google.com/document/d/1f4QII6ioYrXxkfo8nIl4pulZcKYwYetdZBPdwL1Ert4/edit"
              >here</a
            >
            to explore the options available.
          </li>
          <li class="instructionsItem">
            Note that accidently clicking on the map while in the trigger menu
            will place a floor tile.
          </li>
        </ol>
        <h3 class="instructionsTitle instructionsTitle--last">
          Instructions For Downloading
        </h3>
        <ol class="instructionsList">
          <li class="instructionsItem">
            Set the level number as 1 + the current max level in the codebase.
          </li>
          <li class="instructionsItem">
            After downloading, add the level to /core/assets/levels/
          </li>
          <li class="instructionsItem">
            Add an import for this new JSON file in /core/assets/assets.json
          </li>
          <li class="instructionsItem">
            Add a new level object in /core/assets/levels/levels.json
          </li>
          <li class="instructionsItem">
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
import FinishedTriggers from "./FinishedTriggers.vue";
const stringify = require("json-stringify-pretty-compact");

const versionNumber = "1.3";

// types of objects - stored separately so that can be placed on other tiles
const objectTypes = [
  "player",
  "playerLeft",
  "normal",
  "normalLeft",
  "acute",
  "acuteLeft",
  "key",
  "brick",
  "bomb",
];

export default {
  components: { LevelMap, ToolBar, FinishedTriggers },
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
      objects: [],
      mutantPaths: [],
      inputTiles: [],
      inputObjects: [],
      inputPaths: [],
      version: versionNumber,
      dimensions: [0, 0],
      levelLoading: false,
      fileName: "No file chosen",
      triggerList: [],
      triggerToEdit: null,
    };
  },
  // event to retrieve the file name after selected
  mounted() {
    var input = document.getElementsByClassName("fileInput")[0];
    const _this = this;
    input.addEventListener("change", function(e) {
      _this.fileName = e.target.value.split("\\").pop();
    });
  },
  computed: {
    // return true if the current rules of the map are met
    // rules currently require there to be exactly one player and at least one goal tile
    areRulesMet() {
      if (this.tiles.length > 0) {
        let players = this.findObjects("player").concat(
          this.findObjects("playerLeft")
        );
        let goals = this.findObjects("goal");
        return players.length === 1 && goals.length >= 1;
      }
      return false;
    },
    // returns true if a file has been provided but not yet loaded
    fileInputted() {
      return this.fileName !== "No file chosen";
    },
  },
  methods: {
    // update the board dimensions on button click
    updateDimensions() {
      this.width = Number(this.cols);
      this.height = Number(this.rows);
      this.dimensions = [this.width, this.height];
      this.displayMap = true;
    },
    // when the tiles are updated in LevelMap, update them in Dashboard for JSON purposes
    updateTilesAndObjects(tiles, objects, mutantPaths, currentPath) {
      this.tiles = tiles;
      this.objects = objects;
      // do not add current path if it does not contain at least 2 mutants
      if (currentPath.length < 2) {
        this.mutantPaths = mutantPaths;
      } else {
        mutantPaths.push(currentPath);
        this.mutantPaths = mutantPaths;
      }
    },
    // update the tile being placed when a button is clicked on in ToolBar
    updateTilePlacing(type) {
      this.typePlacing = type;
    },
    // add a trigger created from the trigger toolbar
    addTrigger(trigger) {
      this.triggerList.push(trigger);
    },
    // delete a trigger clicked on in the finished triggers
    deleteTrigger(index) {
      this.triggerList = this.triggerList
        .slice(0, index)
        .concat(this.triggerList.slice(index + 1));
    },
    // edit a trigger
    editTrigger(index) {
      this.triggerToEdit = this.triggerList[index];
      this.deleteTrigger(index);
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
        this.fileName = "No file chosen";
      };
      fr.readAsText(files.item(0));
    },
    // converts a loaded in JSON to the parameters needed to display it
    convertJSONToMap(json) {
      // set width and height
      this.width = json.metadata.width;
      this.height = json.metadata.height;
      this.cols = this.width;
      this.rows = this.height;

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
      let objects = new Array(this.height)
        .fill(0)
        .map(() => new Array(this.width).fill("empty"));
      let itemSpawns = json.metadata["item-spawns"];
      for (let i = 0; i < itemSpawns.length; i++) {
        let position = itemSpawns[i].position;
        let type = itemSpawns[i].type.toLowerCase();
        objects[this.height - position[1] - 1][position[0]] = type;
      }

      // copy mutant and paths in
      let mutantSpawns = json.metadata["mutant-spawns"];
      let mutantPaths = [];
      for (let i = 0; i < mutantSpawns.length; i++) {
        let position = mutantSpawns[i].position;
        let type = mutantSpawns[i].type.toLowerCase();
        let direction = mutantSpawns[i].direction ?? "RIGHT";

        if (direction === "LEFT") {
          objects[this.height - position[1] - 1][position[0]] = type + "Left";
        } else {
          objects[this.height - position[1] - 1][position[0]] = type;
        }

        // add patrol path to map, and add mutants along each point after the starting point
        if (mutantSpawns[i]["patrol-path"]) {
          let path = this.transformPathFromJSON(mutantSpawns[i]["patrol-path"]);
          mutantPaths.push(path);
          path.forEach((point, i) => {
            if (i !== 0) {
              if (direction === "LEFT") {
                objects[point[0]][point[1]] = type + "Left";
              } else {
                objects[point[0]][point[1]] = type;
              }
            }
          });
        }
      }

      // set player spawn and direction - default direction if not in JSON is right
      let playerSpawn = json.metadata["player-spawn"];
      let playerDirection = json.metadata["player-direction"] ?? "RIGHT";
      if (playerDirection === "LEFT") {
        objects[this.height - playerSpawn[1] - 1][playerSpawn[0]] =
          "playerLeft";
      } else {
        objects[this.height - playerSpawn[1] - 1][playerSpawn[0]] = "player";
      }

      // set triggerList from json
      // TODO - does position or anything else need to be transformed?
      this.triggerList = json.metadata["triggers"];

      // set input tiles and display the map
      this.inputTiles = tiles;
      this.inputObjects = objects;
      this.inputPaths = mutantPaths;
      this.tiles = this.inputTiles;
      this.objects = this.inputObjects;
      this.mutantPaths = this.inputPaths;

      // set level loading to true so that the dimension watch handles setting tiles/objects properly
      this.levelLoading = true;
      this.dimensions = [this.width, this.height];
      this.displayMap = true;
    },
    // set level loaded to false when done, fixes the loading two levels in a row issue
    setLevelLoaded() {
      this.levelLoading = false;
    },
    // converts the number representation of a tile in the game to the label in the editor
    convertNumberToTile(num) {
      switch (num) {
        case 1:
          return "wall";
        case 2:
          return "glass";
        case 3:
          return "cracked";
        case 4:
          return "goal";
        case 5:
          return "door";
        case 6:
          return "barrel";
        case 7:
          return "grate";
        case 8:
          return "light";
        case 9:
          return "unlock";
        case 10:
          return "open";
        case 11:
          return "speaker";
        case 12:
          return "barrel4";
        case 13:
          return "crate";
        case 14:
          return "crate2";
        case 15:
          return "puddle";
        case 16:
          return "computer";
        case 17:
          return "computerLeft";
        case 18:
          return "computerRight";
        case 19:
          return "computer2";
        case 20:
          return "crate3";
        case 21:
          return "crate4";
        case 22:
          return "stairsUp";
        default:
          return "floor";
      }
    },
    // converts the tile name in the editor to the number representation of a tile in the game
    convertTileToNumber(tile) {
      switch (tile) {
        case "wall":
          return 1;
        case "glass":
          return 2;
        case "cracked":
          return 3;
        case "goal":
          return 4;
        case "door":
          return 5;
        case "barrel":
          return 6;
        case "grate":
          return 7;
        case "light":
          return 8;
        case "unlock":
          return 9;
        case "open":
          return 10;
        case "speaker":
          return 11;
        case "barrel4":
          return 12;
        case "crate":
          return 13;
        case "crate2":
          return 14;
        case "puddle":
          return 15;
        case "computer":
          return 16;
        case "computerLeft":
          return 17;
        case "computerRight":
          return 18;
        case "computer2":
          return 19;
        case "crate3":
          return 20;
        case "crate4":
          return 21;
        case "stairsUp":
          return 22;
        default:
          return 0;
      }
    },
    // create the json based on the map and call the output method
    writeJSON() {
      let json = {};
      json["version"] = this.version;

      let normalMutantSpawns = this.findObjects("normal");
      let acuteMutantSpawns = this.findObjects("acute");
      let normalLeftSpawns = this.findObjects("normalLeft");
      let acuteLeftSpawns = this.findObjects("acuteLeft");
      let mutantSpawns = this.generateMutantSpawns(
        normalMutantSpawns,
        acuteMutantSpawns,
        normalLeftSpawns,
        acuteLeftSpawns,
        this.transformPathsToJSON(this.mutantPaths)
      );

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
        "player-spawn": this.findObjects("player").concat(
          this.findObjects("playerLeft")
        )[0],
        "player-direction":
          this.findObjects("player").length > 0 ? "RIGHT" : "LEFT",
        "mutant-count": mutantSpawns.length,
        "mutant-spawns": mutantSpawns,
        "item-count": itemSpawns.length,
        "item-spawns": itemSpawns,
        triggers: this.triggerList,
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
      let matrix = [];

      // find by looking through object list or tile list depending on which it is
      if (objectTypes.includes(objectType)) {
        matrix = this.objects;
      } else {
        matrix = this.tiles;
      }

      for (let r = 0; r < matrix.length; r++) {
        for (let c = 0; c < matrix[0].length; c++) {
          if (matrix[r][c] === objectType) {
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
    // generate the json representation of mutant spawns based on their type. including mutant paths
    generateMutantSpawns(normal, acute, normalLeft, acuteLeft, paths) {
      let mutantSpawns = [];

      mutantSpawns = mutantSpawns.concat(
        this.generateMutantSpawnsForType(normal, paths, "NORMAL", "RIGHT")
      );
      mutantSpawns = mutantSpawns.concat(
        this.generateMutantSpawnsForType(acute, paths, "ACUTE", "RIGHT")
      );
      mutantSpawns = mutantSpawns.concat(
        this.generateMutantSpawnsForType(normalLeft, paths, "NORMAL", "LEFT")
      );
      mutantSpawns = mutantSpawns.concat(
        this.generateMutantSpawnsForType(acuteLeft, paths, "ACUTE", "LEFT")
      );

      return mutantSpawns;
    },
    // generate mutant spawns from mutant list and path list of given type (normal or acute) and direction (left or right)
    generateMutantSpawnsForType(mutants, paths, type, direction) {
      let spawns = [];

      mutants.forEach((m) => {
        let mutant = {};
        mutant.type = type;
        mutant.direction = direction;
        mutant.position = m;

        // if mutant position matches a starting point, add path information
        // if mutant positions matches a later point, do not include it
        let includeMutant = true;
        paths.forEach((path) => {
          if (path.length > 1) {
            let startingPoint = path[0];
            if (m[0] === startingPoint[0] && m[1] === startingPoint[1]) {
              mutant["patrol-point-count"] = path.length;
              mutant["patrol-path"] = path;
            } else if (this.isArrayInArray(path, m)) {
              includeMutant = false;
            }
          }
        });

        if (includeMutant) {
          spawns.push(mutant);
        }
      });

      return spawns;
    },
    // change all path indices to match json indices
    transformPathsToJSON(paths) {
      let newPaths = [];
      paths.forEach((path) => {
        let newPath = [];
        path.forEach((point) => {
          newPath.push([point[1], this.height - point[0] - 1]);
        });
        newPaths.push(newPath);
      });
      return newPaths;
    },
    // change a singular path's indices from json to map indices
    transformPathFromJSON(path) {
      let newPath = [];
      path.forEach((point) => {
        newPath.push([this.height - point[1] - 1, point[0]]);
      });
      return newPath;
    },
    // returns true if item array is in 2d array arr, otherwise false
    isArrayInArray(arr, item) {
      var itemAsString = JSON.stringify(item);

      var contains = arr.some(function(ele) {
        return JSON.stringify(ele) === itemAsString;
      });
      return contains;
    },
  },
};
</script>

<style scoped>
.dashboard {
  min-width: fit-content;
  background-color: #181818;
}

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
  align-items: center;
  width: fit-content;
  margin-left: auto;
  margin-right: auto;
}

.toolbar {
  margin-left: 5rem;
}

.map {
  margin-right: 5rem;
}

.outputWrapper {
  margin-top: 1rem;
}

.instructions {
  display: flex;
  flex-direction: column;
  justify-content: left;
  text-align: left;
  margin-top: 2rem;
  color: #88a6cd;
  margin-left: 5rem;
}

.instructionsTitle {
  margin-bottom: 0;
  margin-top: 2rem;
}

.instructionsTitle--last {
  margin-top: 1rem;
}

.instructionsList {
  padding-left: 1rem;
}

.instructionsItem {
  margin-top: 5px;
}

.logo {
  margin-top: 60px;
  max-width: 400px;
  border-radius: 10px;
}

input[type="file"] {
  display: none;
}

.fileWrapper {
  border: 1px solid #51616e;
  display: inline-block;
  padding: 6px 12px;
  font-size: 13.3333px;
}

.fileLabelButton {
  background-color: #88a6cd;
  border: 1px solid #88a6cd;
  border-radius: 2px;
  cursor: pointer;
  color: black;
  padding: 1px 8px;
  margin-right: 0.5rem;
}

.fileLabelButton:hover:not([disabled]) {
  background-color: #51616e;
  border-color: #51616e;
}

.finishedTriggers {
  margin: 0 5rem;
}
</style>
