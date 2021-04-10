<template>
  <div class="toolbar">
    <h3>Toolbar</h3>
    <div>
      <div class="section">Tiles</div>
      <!-- // TODO - implement buttons with a v-for -->
      <div class="buttons">
        <div class="item">
          <button
            class="button"
            @click="changePlacing('floor')"
            :disabled="isSelected('floor')"
          >
            <img class="buttonImage" src="@/assets/floor.png" />
          </button>
          <span class="buttonText">Floor</span>
        </div>
        <div class="item">
          <button
            class="button"
            @click="changePlacing('wall')"
            :disabled="isSelected('wall')"
          >
            <img class="buttonImage" src="@/assets/walltop.png" />
          </button>
          <span class="buttonText">Wall</span>
        </div>
        <div class="item">
          <button
            class="button"
            @click="changePlacing('goal')"
            :disabled="isSelected('goal')"
          >
            <img class="buttonImage" src="@/assets/goal.png" />
          </button>
          <span class="buttonText">Goal</span>
        </div>
        <div class="item">
          <button
            class="button"
            @click="changePlacing('barrel')"
            :disabled="isSelected('barrel')"
          >
            <img class="buttonImage" src="@/assets/barrel.png" />
          </button>
          <span class="buttonText">Barrel</span>
        </div>
        <div class="item">
          <button
            class="button"
            @click="changePlacing('grate')"
            :disabled="isSelected('grate')"
          >
            <img class="buttonImage" src="@/assets/floorgrate.png" />
          </button>
          <span class="buttonText">Grate</span>
        </div>
        <div class="item">
          <button
            class="button"
            @click="changePlacing('glass')"
            :disabled="isSelected('glass')"
          >
            <img class="buttonImage" src="@/assets/glass.png" />
          </button>
          <span class="buttonText">Glass</span>
        </div>
        <div class="item">
          <button
            class="button"
            @click="changePlacing('door')"
            :disabled="isSelected('door')"
          >
            <img class="buttonImage" src="@/assets/door.png" />
          </button>
          <span class="buttonText">Door</span>
        </div>
        <div class="item">
          <button
            class="button"
            @click="changePlacing('cracked')"
            :disabled="isSelected('cracked')"
          >
            <img class="buttonImage" src="@/assets/cracked.png" />
          </button>
          <span class="buttonText">Crack</span>
        </div>
      </div>
    </div>
    <div>
      <div class="section">Characters</div>
      <div class="buttons">
        <div class="item">
          <button
            class="button"
            @click="changePlacing('player')"
            :disabled="isSelected('player')"
          >
            <img class="buttonImage" src="@/assets/player.png" />
          </button>
          <span class="buttonText">Player</span>
        </div>
        <div class="item">
          <button
            class="button"
            @click="changePlacing('normal')"
            :disabled="isSelected('normal')"
          >
            <img class="buttonImage" src="@/assets/mutant.png" />
          </button>
          <span class="buttonText">Mutant</span>
        </div>
        <div class="item">
          <button
            class="button"
            @click="changePlacing('acute')"
            :disabled="isSelected('acute')"
          >
            <img class="buttonImage acute" src="@/assets/mutant.png" />
          </button>
          <span class="buttonText">Acute</span>
        </div>
      </div>
      <div class="section">Items</div>
      <div class="buttons">
        <div class="item">
          <button
            class="button"
            @click="changePlacing('key')"
            :disabled="isSelected('key')"
          >
            <img class="buttonImage" src="@/assets/key.png" />
          </button>
          <span class="buttonText">Key</span>
        </div>
        <div class="item">
          <button
            class="button"
            @click="changePlacing('brick')"
            :disabled="isSelected('brick')"
          >
            <img class="buttonImage" src="@/assets/brick.png" />
          </button>
          <span class="buttonText">Brick</span>
        </div>
        <div class="item">
          <button
            class="button"
            @click="changePlacing('bomb')"
            :disabled="isSelected('bomb')"
          >
            <img class="buttonImage" src="@/assets/bomb.png" />
          </button>
          <span class="buttonText">Bomb</span>
        </div>
      </div>
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
    // update what the user can place on mouse click, can be any of the types from the above buttons
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
  width: 11rem;
}
.section {
  margin-top: 1rem;
}
.buttons {
  margin-top: 0.5rem;
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
}
.item {
  display: flex;
  flex-direction: column;
  margin-top: 0.5rem;
}
.button {
  max-width: 40px;
  display: flex;
  justify-content: center;
}
.buttonText {
  margin-top: 0.25rem;
  font-size: 12px;
}
.buttonImage {
  width: 32px;
  height: 32px;
}
.acute {
  filter: brightness(50%);
}
</style>
