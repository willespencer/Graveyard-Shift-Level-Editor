<template>
  <div class="levelmap">
    <div class="row" v-for="(row, r) in height" v-bind:key="r">
      <div
        class="col"
        v-for="(col, c) in width"
        v-bind:key="c"
        :class="{
          floor: isTileType(r, c, 'floor'),
          wall: isTileType(r, c, 'wall'),
        }"
        @click="updateTile(r, c)"
      ></div>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    height: Number,
    width: Number,
    typeToPlace: String,
  },
  data() {
    return {
      tileTypes: [],
    };
  },
  created() {
    this.createTileTypes();
  },
  methods: {
    // create the tile type array, default tiles are set to floor
    createTileTypes() {
      for (let r = 0; r < this.height; r++) {
        this.tileTypes.push([]);
        for (let c = 0; c < this.width; c++) {
          this.tileTypes[r].push("floor");
        }
      }
      this.$emit("tile-changed", this.tileTypes);
    },
    // checks if the tile at r, c is of type type
    isTileType(r, c, type) {
      return this.tileTypes[r][c] === type;
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
  background-color: red;
  height: 60px;
  width: 60px;
}

.floor {
  background-image: url("~@/assets/floor.png");
}

.wall {
  background-image: url("~@/assets/walltop.png");
}
</style>
