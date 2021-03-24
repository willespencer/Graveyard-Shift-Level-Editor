<template>
  <div class="toolbar">
    Toolbar
    <div>
      Tiles
      <button @click="changePlacing('floor')">Floor</button>
      <button @click="changePlacing('wall')">Wall</button>
      <button @click="changePlacing('goal')">Goal</button>
      <button @click="changePlacing('glass')">Glass</button>
    </div>
    <div>
      Objects
      <button @click="changePlacing('player')">Player</button>
      <button @click="changePlacing('mutant')">Mutant</button>
      <button @click="changePlacing('brick')">Brick</button>
      <button @click="changePlacing('bomb')">Bomb</button>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      typePlacing: "floor",
    };
  },
  methods: {
    // update what the user can place on mouse click
    // type can be "floor", "wall", "goal", "player", "mutant", "brick", or "bomb"
    changePlacing(type) {
      this.typePlacing = type;
      this.$emit("update-tile-placing", type);
    },
  },
};
</script>

<style scoped></style>
