<template>
  <div class="triggers">
    <h4>Add Trigger</h4>
    <div class="field">
      <div class="section">Type</div>
      <div class="buttons">
        <button
          v-for="(bool, option) in typeButtons"
          :key="option"
          class="button"
          @click="selectOption(typeButtons, option)"
          :disabled="isSelected(typeButtons, option)"
        >
          {{ option }}
        </button>
      </div>
    </div>
    <div class="field">
      <div class="section">Action</div>
      <div class="buttons">
        <button
          v-for="(bool, option) in actionButtons"
          :key="option"
          class="button"
          @click="selectOption(actionButtons, option)"
          :disabled="isSelected(actionButtons, option)"
        >
          {{ option }}
        </button>
      </div>
    </div>
    <div class="field">
      <div class="section">Trigger Class</div>
      <div class="buttons">
        <button
          v-for="(bool, option) in classButtons"
          :key="option"
          class="button"
          @click="selectOption(classButtons, option)"
          :disabled="isSelected(classButtons, option)"
        >
          {{ option }}
        </button>
      </div>
    </div>
    <div class="field">
      <div class="section">Item Type</div>
      <div class="buttons">
        <button
          v-for="(bool, option) in itemButtons"
          :key="option"
          class="button"
          @click="selectOption(itemButtons, option)"
          :disabled="isSelected(itemButtons, option)"
        >
          {{ option }}
        </button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    let typeButtons = {
      Dialogue: false,
      Control: false,
    };
    let actionButtons = {
      Enter: false,
      Pickup: false,
      Use: false,
      Use_Enter: false,
      Drop: false,
      Drop_Enter: false,
    };
    let classButtons = {
      Player: false,
      Mutant: false,
      Brick: false,
      Bomb: false,
      Key: false,
    };
    let itemButtons = {
      Brick: false,
      Bomb: false,
      Key: false,
      Use: false,
    };
    return {
      typeButtons,
      actionButtons,
      classButtons,
      itemButtons,
    };
  },
  methods: {
    // set the key equivalent to the option string to true in list, set all others to false
    selectOption(list, option) {
      for (const key in list) {
        if (key === option) {
          list[key] = true;
        } else {
          list[key] = false;
        }
      }
    },
    // returns the boolean at key option in list
    isSelected(list, option) {
      return list[option];
    },
  },
};
</script>

<style scoped>
.field {
  margin-top: 0.5rem;
}
.buttons {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
}
.button {
  display: flex;
  justify-content: center;
  align-items: center;
  max-width: 80px;
  min-height: 22px;
  flex-basis: 50%;
  margin-top: 0.5rem;
}
.button:hover {
  color: rgba(16, 16, 16, 0.3);
}
</style>
