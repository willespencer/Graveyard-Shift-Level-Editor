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
          v-if="!isTile(r, c)"
          :src="getForegroundImage(r, c)"
          class="image"
          :class="{ acute: isTileType(r, c, 'acute') }"
        />
      </div>
    </div>
  </div>
</template>

<script>
// TODO update with more types as they get added to the game

// types of tiles that show up by themselves
const tileTypes = ["floor", "wall", "goal", "grate", "cracked"];

// types of tiles that can act as walls for doors/glass to attach to
const wallTypes = ["wall", "glass", "cracked", "goal", "door"];

// images that show up in as a background image / tile
import floorImage from "@/assets/floor.png";
import wallImage from "@/assets/walltop.png";
import goalImage from "@/assets/goal.png";
import grateImage from "@/assets/floorgrate.png";
import crackedImage from "@/assets/cracked.png";

// other types of walls
import wallTop from "@/assets/walltopshort.png";
import wallSide from "@/assets/wallside.png";

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
    editedMap: Boolean,
  },
  data() {
    // if tiles inputted (i.e. map loaded), display that instead of the default map
    let tiles = [];
    let tempImage = playerImage;
    if (this.inputTiles.length > 0) {
      tiles = this.inputTiles;
    }
    return {
      tileTypes: tiles,
      isMouseDown: false,
      tempImage,
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
        }

        let newTiles = [];
        for (let r = 0; r < val[1]; r++) {
          newTiles.push([]);
          for (let c = 0; c < val[0]; c++) {
            // place a wall if it is an edge of the map, the existing tile if it is in the old range, otherwise place the floor
            if (r === 0 || r === val[1] - 1) {
              newTiles[r].push("wall");
            } else if (c === 0 || c === val[0] - 1) {
              newTiles[r].push("wall");
            } else if (r < oldVal[1] && c < oldVal[0]) {
              newTiles[r].push(this.tileTypes[r][c]);
            } else {
              newTiles[r].push("floor");
            }
          }
        }

        this.tileTypes = newTiles;
        this.$emit("tile-changed", this.tileTypes);
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
        for (let c = 0; c < this.width; c++) {
          if (r === 0 || r === this.height - 1) {
            this.tileTypes[r].push("wall");
          } else if (c === 0 || c === this.width - 1) {
            this.tileTypes[r].push("wall");
          } else {
            this.tileTypes[r].push("floor");
          }
        }
      }
      this.$emit("tile-changed", this.tileTypes);
    },
    // checks if the tile at r, c is of type type
    isTileType(r, c, type) {
      return this.tileTypes[r][c] === type;
    },
    // return true if a type is a tile
    isTile(r, c) {
      return tileTypes.includes(this.tileTypes[r][c]);
    },
    // when clicked, update the tile at r, c to whatever type of tile is being placed
    updateTile(r, c) {
      const newRow = this.tileTypes[r].slice(0);
      newRow[c] = this.typeToPlace;
      this.$set(this.tileTypes, r, newRow);
      this.$emit("tile-changed", this.tileTypes);
    },
    // gets the src of applicable tiles. If there is an object on this space, returns a floor
    getBackgroundImage(r, c) {
      if (this.isTileType(r, c, "wall")) {
        return this.getWallImage(r, c);
      } else if (this.isTileType(r, c, "goal")) {
        return goalImage;
      } else if (this.isTileType(r, c, "grate")) {
        return grateImage;
      } else if (this.isTileType(r, c, "cracked")) {
        return crackedImage;
      } else {
        return floorImage;
      }
    },
    // for objects or tiles that show up in front of the floor, get the image src
    getForegroundImage(r, c) {
      if (this.isTileType(r, c, "player")) {
        return playerImage;
      } else if (this.isTileType(r, c, "normal")) {
        return mutantImage;
      } else if (this.isTileType(r, c, "acute")) {
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
        !this.isTileType(r + 1, c, "wall")
      ) {
        return wallSide;
      } else if (r - 1 >= 0 && this.isTileType(r - 1, c, "floor")) {
        return wallTop;
      } else {
        return wallImage;
      }
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
}

.image {
  max-width: 45px;
  max-height: 45px;
}

.acute {
  filter: brightness(50%);
}
</style>
