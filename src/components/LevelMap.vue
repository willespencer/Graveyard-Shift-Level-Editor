<template>
  <div class="levelmap">
    <div class="row" v-for="(row, r) in height" v-bind:key="r">
      <div
        class="col"
        v-for="(col, c) in width"
        v-bind:key="c"
        :style="{ backgroundImage: 'url(' + getBackgroundImage(r, c) + ')' }"
        @mousedown="down(r, c)"
        @mousemove="move(r, c)"
        @mouseup="up()"
      >
        <img
          v-if="!isTile(r, c) || isTopWall(r, c)"
          :src="getForegroundImage(r, c)"
          class="image"
          :class="{
            acute: isTileType(r, c, 'acute') || isTileType(r, c, 'acuteLeft'),
            wall: isTileType(r, c, 'wall') || isTileType(r, c, 'light'),
            topWall: isTopWall(r, c),
            flipped: isFlipped(r, c),
          }"
        />
      </div>
    </div>
  </div>
</template>

<script>
// TODO update with more types as they get added to the game

// types of tiles that show up by themselves. Excludes walls because of other wall types
const foregroundList = ["floor", "goal", "grate", "cracked"];

// types of tiles that can act as walls for doors/glass to attach to
const wallTypes = ["wall", "glass", "cracked", "goal", "door", "light"];

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
// types of tiles that can be placed on top of
const pureTiles = ["floor", "grate"];

// images that show up in as a background image / tile
import floorImage from "@/assets/floor.png";
import wallImage from "@/assets/walltop.png";
import goalImage from "@/assets/goal.png";
import grateImage from "@/assets/floorgrate.png";
import crackedImage from "@/assets/cracked.png";

// other types of walls
import wallTop from "@/assets/walltopshort.png";
import wallSide from "@/assets/wallside.png";
import wallLight from "@/assets/wall_light.png";

// other images that show up in front of floor tiles (including objects, barrels, doors, etc.)
import playerImage from "@/assets/player.png";
import mutantImage from "@/assets/mutant.png";
import barrelImage from "@/assets/barrel.png";
import doorImage from "@/assets/door.png";
import doorSideImage from "@/assets/door_side.png";
import glassImage from "@/assets/glass.png";
import glassSideImage from "@/assets/glass_side.png";
import brickImage from "@/assets/brick.png";
import bombImage from "@/assets/bomb.png";
import keyImage from "@/assets/key.png";

export default {
  props: {
    dimensions: Array,
    typeToPlace: String,
    inputTiles: Array,
    inputObjects: Array,
    levelLoading: Boolean,
  },
  data() {
    // if tiles inputted (i.e. map loaded), display that instead of the default map
    let tiles = [];
    let objects = [];
    if (this.inputTiles.length > 0 && this.inputObjects.length > 0) {
      tiles = this.inputTiles;
      objects = this.inputObjects;
    }
    return {
      tileTypes: tiles,
      isMouseDown: false,
      objects,
    };
  },
  created() {
    this.createTileTypes();
  },
  computed: {
    width() {
      return this.dimensions[0];
    },
    height() {
      return this.dimensions[1];
    },
  },
  // if size was edited, update tile types
  watch: {
    dimensions: {
      immediate: true,
      handler(val, oldVal) {
        // if this is the first time dimensions are set, ignore handler
        if (!oldVal) {
          return;
          // if dimensions have been set before but level is being loaded in, set tiles/objects as input
        } else if (this.levelLoading) {
          this.tileTypes = this.inputTiles;
          this.objects = this.inputObjects;
          this.$emit("level-loaded");
          return;
        }

        let newTiles = [];
        let newObjects = [];
        for (let r = 0; r < val[1]; r++) {
          newTiles.push([]);
          newObjects.push([]);
          for (let c = 0; c < val[0]; c++) {
            // place a wall if it is an edge of the map, the existing tile if it is in the old range, otherwise place the floor
            if (r === 0 || r === val[1] - 1) {
              newTiles[r].push("wall");
              newObjects[r].push("empty");
            } else if (c === 0 || c === val[0] - 1) {
              newTiles[r].push("wall");
              newObjects[r].push("empty");
            } else if (r < oldVal[1] && c < oldVal[0]) {
              newTiles[r].push(this.tileTypes[r][c]);
              newObjects[r].push(this.objects[r][c]);
            } else {
              newTiles[r].push("floor");
              newObjects[r].push("empty");
            }
          }
        }

        this.tileTypes = newTiles;
        this.objects = newObjects;
        this.$emit("tile-changed", this.tileTypes, this.objects);
      },
    },
  },
  methods: {
    down(r, c) {
      this.isMouseDown = true;
      this.updateTile(r, c);
    },
    move(r, c) {
      if (this.isMouseDown) {
        this.updateTile(r, c);
      }
    },
    up() {
      this.isMouseDown = false;
    },
    // create the tile type array, default tiles are set to wall around the border, floor otherwise
    createTileTypes() {
      for (let r = 0; r < this.height; r++) {
        this.tileTypes.push([]);
        this.objects.push([]);
        for (let c = 0; c < this.width; c++) {
          if (r === 0 || r === this.height - 1) {
            this.tileTypes[r].push("wall");
          } else if (c === 0 || c === this.width - 1) {
            this.tileTypes[r].push("wall");
          } else {
            this.tileTypes[r].push("floor");
          }
          // no matter what the tile is, set no object to be on top of it
          this.objects[r].push("empty");
        }
      }
      this.$emit("tile-changed", this.tileTypes, this.objects);
    },
    // checks if the tile at r, c is of type type, or if there is an object at r, c, if the object matches type
    isTileType(r, c, type) {
      let tile = this.tileTypes[r][c];
      let object = this.objects[r][c];
      if (object === "empty") {
        return tile === type;
      } else {
        return object === type;
      }
    },
    // return true if a type is a tile that shows up in the foreground and there is no object
    isTile(r, c) {
      return (
        foregroundList.includes(this.tileTypes[r][c]) &&
        this.objects[r][c] === "empty"
      );
    },
    // returns true if a tile's image should be flipped
    isFlipped(r, c) {
      return this.objects[r][c].includes("Left");
    },
    // when clicked, update the tile at r, c to whatever type of tile is being placed
    updateTile(r, c) {
      // if it is an object, update object types (and switch background tile if needed)
      if (objectTypes.includes(this.typeToPlace)) {
        const newObjectRow = this.objects[r].slice(0);
        newObjectRow[c] = this.typeToPlace;
        this.$set(this.objects, r, newObjectRow);

        // switch background to floor if not a pureType, otherwise keep it
        if (!pureTiles.includes(this.tileTypes[r][c])) {
          const newRow = this.tileTypes[r].slice(0);
          newRow[c] = "floor";
          this.$set(this.tileTypes, r, newRow);
        }
      } else {
        // if not an object, set object to null and floor to type
        const newRow = this.tileTypes[r].slice(0);
        newRow[c] = this.typeToPlace;
        this.$set(this.tileTypes, r, newRow);

        const newObjectRow = this.objects[r].slice(0);
        newObjectRow[c] = "empty";
        this.$set(this.objects, r, newObjectRow);
      }
      this.$emit("tile-changed", this.tileTypes, this.objects);
    },
    // gets the src of applicable tiles. If there is an object on this space, return whichever tile is below it
    getBackgroundImage(r, c) {
      if (this.isTileType(r, c, "goal")) {
        return goalImage;
      } else if (
        this.isTileType(r, c, "grate") ||
        (this.objects[r][c] !== "empty" && this.tileTypes[r][c] === "grate")
      ) {
        return grateImage;
      } else if (this.isTileType(r, c, "cracked")) {
        return crackedImage;
      } else if (
        this.isTileType(r, c, "wall") ||
        this.isTileType(r, c, "light")
      ) {
        return wallImage;
      } else {
        return floorImage;
      }
    },
    // for objects or tiles that show up in front of the floor, get the image src
    getForegroundImage(r, c) {
      if (
        this.isTileType(r, c, "player") ||
        this.isTileType(r, c, "playerLeft")
      ) {
        return playerImage;
      } else if (this.isTileType(r, c, "wall")) {
        return this.getWallImage(r, c);
      } else if (this.isTileType(r, c, "light")) {
        return wallLight;
      } else if (
        this.isTileType(r, c, "normal") ||
        this.isTileType(r, c, "normalLeft")
      ) {
        return mutantImage;
      } else if (
        this.isTileType(r, c, "acute") ||
        this.isTileType(r, c, "acuteLeft")
      ) {
        return mutantImage;
      } else if (this.isTileType(r, c, "bomb")) {
        return bombImage;
      } else if (this.isTileType(r, c, "key")) {
        return keyImage;
      } else if (this.isTileType(r, c, "brick")) {
        return brickImage;
      } else if (this.isTileType(r, c, "barrel")) {
        return barrelImage;
      } else if (this.isTileType(r, c, "door")) {
        if (this.shouldDisplaySide(r, c)) {
          return doorSideImage;
        } else {
          return doorImage;
        }
      } else if (this.isTileType(r, c, "glass")) {
        if (this.shouldDisplaySide(r, c)) {
          return glassSideImage;
        } else {
          return glassImage;
        }
      } else if (this.isTileType(r, c, "floor")) {
        return wallTop;
      }

      return null;
    },
    // returns true for objects that should show up in their side profile form like glass and doors
    // this specifically occurs if a non-floor tile is above or below this object
    shouldDisplaySide(r, c) {
      // true if the tile abvoe is in bounds and is a wall type
      if (r - 1 >= 0 && wallTypes.includes(this.tileTypes[r - 1][c])) {
        return true;
        // true if the tile below is in bounds and a wall type
      } else if (
        r + 1 < this.tileTypes.length &&
        wallTypes.includes(this.tileTypes[r + 1][c])
      ) {
        return true;
      }

      return false;
    },
    // returns the wall image depending on the surrounding tiles per game logic
    getWallImage(r, c) {
      if (
        r + 1 >= this.tileTypes.length ||
        (!this.isTileType(r + 1, c, "wall") &&
          !this.isTileType(r + 1, c, "light"))
      ) {
        return wallSide;
      } else {
        return wallImage;
      }
    },
    // returns true if the floor should display a top wall image on the bottom
    isTopWall(r, c) {
      return (
        r + 1 < this.tileTypes.length &&
        this.isTileType(r, c, "floor") &&
        (this.isTileType(r + 1, c, "wall") ||
          this.isTileType(r + 1, c, "light"))
      );
    },
  },
};
</script>

<style scoped>
.levelmap {
  margin-top: 1rem;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.row {
  display: flex;
}

.col {
  height: 45px;
  width: 45px;
  background-size: cover;
  position: relative;

  user-drag: none;
  user-select: none;
  -moz-user-select: none;
  -webkit-user-drag: none;
  -webkit-user-select: none;
  -ms-user-select: none;
}

.image {
  max-width: 45px;
  max-height: 45px;
  user-drag: none;
  user-select: none;
  -moz-user-select: none;
  -webkit-user-drag: none;
  -webkit-user-select: none;
  -ms-user-select: none;
}

.acute {
  filter: brightness(50%);
}

.wall {
  padding-top: 12px;
  position: absolute;
  left: 0;
  z-index: 2;
}

.topWall {
  padding-top: 34px;
  position: absolute;
  left: 0;
  z-index: 2;
}

.flipped {
  transform: scaleX(-1);
}
</style>
