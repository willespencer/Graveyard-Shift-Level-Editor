<template>
  <div class="toolbar">
    <h3>Toolbar</h3>
    <div v-if="isSelected('trigger')">
      <triggers
        :triggerToEdit="triggerToEdit"
        :currentTriggerPosition="currentTriggerPosition"
        @add-trigger="addTrigger"
        @close-triggers="closeTriggerMenu"
        @current-trigger="setCurrentTrigger"
      />
    </div>
    <div v-else>
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
            @click="changePlacing('light')"
            :disabled="isSelected('light')"
          >
            <img class="buttonImage" src="@/assets/wall_light.png" />
          </button>
          <span class="buttonText">Light</span>
        </div>
        <div class="item">
          <button
            class="button"
            @click="changePlacing('stairsUp')"
            :disabled="isSelected('stairsUp')"
          >
            <img class="buttonImage" src="@/assets/stairs_up.png" />
          </button>
          <span class="buttonText">Stairs</span>
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
            @click="changePlacing('unlock')"
            :disabled="isSelected('unlock')"
          >
            <img class="buttonImage" src="@/assets/door_unlocked.png" />
          </button>
          <span class="buttonText">Unlock</span>
        </div>
        <div class="item">
          <button
            class="button"
            @click="changePlacing('open')"
            :disabled="isSelected('open')"
          >
            <img class="buttonImage" src="@/assets/door_side_unlocked.png" />
          </button>
          <span class="buttonText">Open</span>
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
            @click="changePlacing('speaker')"
            :disabled="isSelected('speaker')"
          >
            <img class="buttonImage" src="@/assets/speaker.png" />
          </button>
          <span class="buttonText">Speaker</span>
        </div>
        <div class="item">
          <button
            class="button"
            @click="changePlacing('puddle')"
            :disabled="isSelected('puddle')"
          >
            <img class="buttonImage" src="@/assets/puddle.png" />
          </button>
          <span class="buttonText">Puddle</span>
        </div>
        <div class="item">
          <button
            class="button"
            @click="changePlacing('checkpoint')"
            :disabled="isSelected('checkpoint')"
          >
            <img class="buttonImage" src="@/assets/checkpoint.png" />
          </button>
          <span class="buttonText">Check-point</span>
        </div>
      </div>
      <div class="section">Aesthetics</div>
      <div class="buttons">
        <div class="item">
          <button
            class="button"
            @click="changePlacing('barrel')"
            :disabled="isSelected('barrel')"
          >
            <img class="buttonImage" src="@/assets/barrel3.png" />
          </button>
          <span class="buttonText">Barrel 3</span>
        </div>
        <div class="item">
          <button
            class="button"
            @click="changePlacing('barrel4')"
            :disabled="isSelected('barrel4')"
          >
            <img class="buttonImage" src="@/assets/barrel4.png" />
          </button>
          <span class="buttonText">Barrel 4</span>
        </div>
        <div class="item">
          <button
            class="button"
            @click="changePlacing('crate')"
            :disabled="isSelected('crate')"
          >
            <img class="buttonImage" src="@/assets/crate.png" />
          </button>
          <span class="buttonText">Crate 1</span>
        </div>
        <div class="item">
          <button
            class="button"
            @click="changePlacing('crate2')"
            :disabled="isSelected('crate2')"
          >
            <img class="buttonImage" src="@/assets/crate2.png" />
          </button>
          <span class="buttonText">Crate 2</span>
        </div>
        <div class="item">
          <button
            class="button"
            @click="changePlacing('crate3')"
            :disabled="isSelected('crate3')"
          >
            <img class="buttonImage" src="@/assets/crate3.png" />
          </button>
          <span class="buttonText">Crate 3</span>
        </div>
        <div class="item">
          <button
            class="button"
            @click="changePlacing('crate4')"
            :disabled="isSelected('crate4')"
          >
            <img class="buttonImage" src="@/assets/crate4.png" />
          </button>
          <span class="buttonText">Crate 4</span>
        </div>
        <div class="item">
          <button
            class="button"
            @click="changePlacing('computer')"
            :disabled="isSelected('computer')"
          >
            <img class="buttonImage" src="@/assets/computer.png" />
          </button>
          <span class="buttonText">PC</span>
        </div>
        <div class="item">
          <button
            class="button"
            @click="changePlacing('computerLeft')"
            :disabled="isSelected('computerLeft')"
          >
            <img class="buttonImage" src="@/assets/computer_side.png" />
          </button>
          <span class="buttonText">PC (Left)</span>
        </div>
        <div class="item">
          <button
            class="button"
            @click="changePlacing('computerRight')"
            :disabled="isSelected('computerRight')"
          >
            <img class="buttonImage" src="@/assets/computer_side_right.png" />
          </button>
          <span class="buttonText">PC (Right)</span>
        </div>
        <div class="item">
          <button
            class="button"
            @click="changePlacing('computer2')"
            :disabled="isSelected('computer2')"
          >
            <img class="buttonImage" src="@/assets/computer2.png" />
          </button>
          <span class="buttonText">PC 2</span>
        </div>
      </div>
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
          <span class="buttonText">Player (Right)</span>
        </div>
        <div class="item">
          <button
            class="button"
            @click="changePlacing('normal')"
            :disabled="isSelected('normal')"
          >
            <img class="buttonImage" src="@/assets/mutant.png" />
          </button>
          <span class="buttonText">Mutant (Right)</span>
        </div>
        <div class="item">
          <button
            class="button"
            @click="changePlacing('acute')"
            :disabled="isSelected('acute')"
          >
            <img class="buttonImage acute" src="@/assets/mutant.png" />
          </button>
          <span class="buttonText">Acute (Right)</span>
        </div>
        <div class="item">
          <button
            class="button"
            @click="changePlacing('playerLeft')"
            :disabled="isSelected('playerLeft')"
          >
            <img class="buttonImage flipped" src="@/assets/player.png" />
          </button>
          <span class="buttonText">Player (Left)</span>
        </div>
        <div class="item">
          <button
            class="button"
            @click="changePlacing('normalLeft')"
            :disabled="isSelected('normalLeft')"
          >
            <img class="buttonImage flipped" src="@/assets/mutant.png" />
          </button>
          <span class="buttonText">Mutant (Left)</span>
        </div>
        <div class="item">
          <button
            class="button"
            @click="changePlacing('acuteLeft')"
            :disabled="isSelected('acuteLeft')"
          >
            <img class="buttonImage acute flipped" src="@/assets/mutant.png" />
          </button>
          <span class="buttonText">Acute (Left)</span>
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
      <div class="section">Customization</div>
      <div class="buttons">
        <div class="item">
          <button
            class="button"
            @click="changePlacing('path')"
            :disabled="isSelected('path')"
          >
            <img class="buttonImage flipped" src="@/assets/left_arrow.png" />
          </button>
          <span class="buttonText">Mutant Path</span>
        </div>
        <div class="item">
          <button
            class="button"
            @click="changePlacing('trigger')"
            :disabled="isSelected('trigger')"
          >
            <img class="buttonImage" src="@/assets/32x32.png" />
          </button>
          <span class="buttonText">Triggers</span>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Triggers from "@/components/Triggers.vue";

export default {
  components: { Triggers },
  props: {
    triggerToEdit: Object,
    currentTriggerPosition: Array,
  },
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
    addTrigger(trigger) {
      this.$emit("add-trigger", trigger);
    },
    // change the placing tile to floor if trigger menu closed and emit
    closeTriggerMenu() {
      this.changePlacing("floor");
      this.$emit("close-triggers");
    },
    // return the json of the current trigger being edited
    setCurrentTrigger(trigger) {
      this.$emit("set-current-trigger", trigger);
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
  align-items: center;
  margin-top: 0.5rem;
  flex-basis: 33.333333%;
}
.button {
  max-width: 40px;
  display: flex;
  justify-content: center;
}
.buttonText {
  margin-top: 0.25rem;
  font-size: 12px;
  max-width: 42px;
}
.buttonImage {
  width: 32px;
  height: 32px;
}
.acute {
  filter: brightness(50%);
}
.flipped {
  transform: scaleX(-1);
}
</style>
