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
        }"
        @mousedown="down(r, c)"
        @mousemove="move(r, c)"
        @mouseup="up()"
      >
        <div
          v-if="!isTile(r, c)"
          class="image"
          :class="{
            player: isTileType(r, c, 'player'),
            normal: isTileType(r, c, 'normal'),
            acute: isTileType(r, c, 'acute'),
            bomb: isTileType(r, c, 'bomb'),
            brick: isTileType(r, c, 'brick'),
            key: isTileType(r, c, 'key'),
          }"
        />
      </div>
    </div>
  </div>
</template>

<script>
// TODO update with more types as they get added to the game
const tileTypes = ["floor", "wall", "glass", "goal", "door"];

export default {
  props: {
    height: Number,
    width: Number,
    typeToPlace: String,
    inputTiles: Array,
  },
  data() {
    // if tilea inputted (i.e. map loaded), display that instead of the default map
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
  height: 60px;
  width: 60px;
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

.door {
  background-image: url("~@/assets/door.png");
}

.goal {
  background-image: url("~@/assets/goal.png");
}

.image {
  width: 60px;
  height: 60px;
}

.player {
  background-image: url("~@/assets/player.png");
}

.normal {
  background-image: url("~@/assets/mutant.png");
}

.acute {
  background-image: url("~@/assets/mutant.png");
  filter: brightness(50%);
}

.brick {
  background-image: url("~@/assets/brick.png");
}

.bomb {
  background-image: url("~@/assets/bomb.png");
}

.key {
  background-image: url("~@/assets/key.png");
}
</style>
