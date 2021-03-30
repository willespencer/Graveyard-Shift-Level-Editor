<template>
  <div class="toolbar">
    <h3>Toolbar</h3>
    <div>
      <div class="section">Tiles</div>
      <div class="buttons">
        <button
          class="button"
          @click="changePlacing('floor')"
          :disabled="isSelected('floor')"
        >
          Floor
        </button>
        <button
          class="button"
          @click="changePlacing('wall')"
          :disabled="isSelected('wall')"
        >
          Wall
        </button>
        <button
          class="button"
          @click="changePlacing('goal')"
          :disabled="isSelected('goal')"
        >
          Goal
        </button>
        <button
          class="button"
          @click="changePlacing('grate')"
          :disabled="isSelected('grate')"
        >
          Grate
        </button>
        <button
          class="button"
          @click="changePlacing('glass')"
          :disabled="isSelected('glass')"
        >
          Glass
        </button>
        <button
          class="button"
          @click="changePlacing('door')"
          :disabled="isSelected('door')"
        >
          Door
        </button>
      </div>
    </div>
    <div>
      <div class="section">Objects</div>
      <div class="buttons">
        <button
          class="button"
          @click="changePlacing('player')"
          :disabled="isSelected('player')"
        >
          Player
        </button>
        <button
          class="button"
          @click="changePlacing('normal')"
          :disabled="isSelected('normal')"
        >
          Normal Mutant
        </button>
        <button
          class="button"
          @click="changePlacing('acute')"
          :disabled="isSelected('acute')"
        >
          Acute Mutant
        </button>
        <button
          class="button"
          @click="changePlacing('brick')"
          :disabled="isSelected('brick')"
        >
          Brick
        </button>
        <button
          class="button"
          @click="changePlacing('bomb')"
          :disabled="isSelected('bomb')"
        >
          Bomb
        </button>
        <button
          class="button"
          @click="changePlacing('key')"
          :disabled="isSelected('key')"
        >
          Key
        </button>
      </div>
    </div>
    <div>
      <div class="section">Rules</div>
      <ol>
        <li>There must be exactly 1 player on the level</li>
        <li>There must be exactly 1 goal on the level</li>
      </ol>
      <span>If these rules are not followed, level cannot be downloaded!</span>
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
    // type can be "floor", "wall", "goal", "door", "player", "normal", "acute", "brick", "bomb", or "key"
    changePlacing(type) {
      this.typePlacing = type;
      this.$emit("update-tile-placing", type);
    },
    // true if the type of the button is the same as the type of tile being placed
    // used to determine which button is disabled
    isSelected(type) {
      return type == this.typePlacing;
    },
  },
};
</script>

<style scoped>
.toolbar {
  margin-right: 3rem;
}
.section {
  margin-top: 1rem;
}
.buttons {
  margin-top: 0.5rem;
}
.button {
  margin-left: 1rem;
  margin-top: 0.5rem;
}
.button:first-child {
  margin-left: 0;
}
</style>
