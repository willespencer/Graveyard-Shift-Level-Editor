<template>
  <div class="levelmap">
    <div class="row" v-for="(row, r) in height" v-bind:key="r">
      <div
        class="col"
        v-for="(col, c) in width"
        v-bind:key="c"
        :class="{
          floor: isTileType(r, c, 'floor') || !isTile(r, c),
          wall: isTileType(r, c, 'wall'),
          glass: isTileType(r, c, 'glass'),
          door: isTileType(r, c, 'door'),
          goal: isTileType(r, c, 'goal'),
          grate: isTileType(r, c, 'grate'),
          cracked: isTileType(r, c, 'cracked'),
        }"
        @mousedown="down(r, c)"
        @mousemove="move(r, c)"
        @mouseup="up()"
      >
        <img
          v-if="!isTile(r, c)"
          :src="getImage(r, c)"
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
const tileTypes = [
  "floor",
  "wall",
  "glass",
  "goal",
  "door",
  "grate",
  "cracked",
];

// other images that show up in front of floor tiles (including objects, barrels, etc.)
import playerImage from "@/assets/player.png";
import mutantImage from "@/assets/mutant.png";
import barrelImage from "@/assets/barrel.png";
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
    if (this.inputTiles.length > 0) {
      tiles = this.inputTiles;
    }
    return {
      tileTypes: tiles,
      isMouseDown: false,
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
    // for objects or tiles that show up in front of the floor, get the image src
    getImage(r, c) {
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
      }

      return null;
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

.floor {
  background-image: url("~@/assets/floor.png");
}

.wall {
  background-image: url("~@/assets/walltop.png");
}

.glass {
  background-image: url("~@/assets/glass.png");
}

.cracked {
  background-image: url("~@/assets/cracked.png");
}

.door {
  background-image: url("~@/assets/door.png");
}

.goal {
  background-image: url("~@/assets/goal.png");
}

.grate {
  background-image: url("~@/assets/floorgrate.png");
}

.image {
  max-width: 45px;
  max-height: 45px;
}

.acute {
  filter: brightness(50%);
}
</style>
