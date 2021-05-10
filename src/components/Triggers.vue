<template>
  <div class="triggers">
    <h4>Create a New Trigger</h4>
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
    <div class="field">
      <div class="section">Position</div>
      <div class="sizeInputs">
        <input class="input" v-model="positionX" /> x
        <input class="input" v-model="positionY" />
      </div>
    </div>
    <div class="field">
      <div class="section">Dimensions</div>
      <div class="sizeInputs">
        <input class="input" v-model="dimensionsX" /> x
        <input class="input" v-model="dimensionsY" />
      </div>
    </div>
    <div class="field">
      <div class="section">Repetitions</div>
      <div class="sizeInputs">
        <div class="repetitionWrapper">
          <input class="repetitionInput" v-model="repetitionsMin" />
          <span class="inputLabel">Min</span>
        </div>
        <div class="repetitionWrapper">
          <input class="repetitionInput" v-model="repetitionsMax" />
          <span class="inputLabel">Max</span>
        </div>
      </div>
    </div>
    <div
      class="field"
      :class="{ isDisabled: !isSelected(typeButtons, 'Control') }"
    >
      <div class="section">
        Display
      </div>
      <span class="subSection">(Control Only)</span>
      <div class="sizeInputs">
        <input
          class="input"
          v-model="displayX"
          :disabled="!isSelected(typeButtons, 'Control')"
        />
        x
        <input
          class="input"
          v-model="displayY"
          :disabled="!isSelected(typeButtons, 'Control')"
        />
      </div>
    </div>
    <div class="field">
      <div class="section">
        <span v-if="isSelected(typeButtons, 'Control')">Control </span>Text
      </div>
      <div v-for="index in textInputs.length" :key="index">
        <div v-if="isCurrentPage(index)">
          <textarea
            class="textArea"
            v-model="textInputs[index - 1]"
            rows="7"
          ></textarea>
          <div class="bottomText" v-if="isSelected(typeButtons, 'Dialogue')">
            <span class="pageText">Dialogue {{ index }}</span>
            <div>
              <button
                @click="prevPage()"
                :disabled="index <= 1"
                class="pageButton"
              >
                Prev
              </button>
              <button
                @click="nextPage()"
                :disabled="!textInputs[index - 1]"
                class="pageButton"
              >
                Next
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
    <div class="bottom">
      <button class="button" @click="addTrigger()" :disabled="!canAddTrigger">
        Add Trigger
      </button>
      <button class="button returnButton" @click="returnToToolbar()">
        Return to Toolbar
      </button>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    triggerToEdit: Object,
  },
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
      None: false,
    };
    return {
      typeButtons,
      actionButtons,
      classButtons,
      itemButtons,
      positionX: 0,
      positionY: 0,
      dimensionsX: 0,
      dimensionsY: 0,
      repetitionsMin: "",
      repetitionsMax: "",
      displayX: "",
      displayY: "",
      textInputs: [""],
      currentTextPage: 1,
    };
  },
  watch: {
    triggerToEdit: {
      immediate: true,
      handler(val) {
        if (!val) {
          return;
        }
        // if a trigger is edited, clear existing options and load in the data
        this.clearData();

        this.selectOption(
          this.actionButtons,
          this.uppercaseFirstLetter(val.action)
        );
        this.selectOption(
          this.typeButtons,
          this.uppercaseFirstLetter(val.type)
        );

        let trigger = "Player";
        if (val["trigger-class"] && val["trigger-class"] !== "") {
          trigger = this.uppercaseFirstLetter(val["trigger-class"]);
        }
        this.selectOption(this.classButtons, trigger);

        let item = "None";
        if (val["item-type"] && val["item-type"] !== "") {
          item = this.uppercaseFirstLetter(val["item-type"]);
        }
        this.selectOption(this.itemButtons, item);

        this.positionX = val.position[0];
        this.positionY = val.position[1];

        this.dimensionsX = val.dimensions[0];
        this.dimensionsY = val.dimensions[1];

        if (val["min-repetitions"]) {
          this.repetitionsMin = val["min-repetitions"];
        }

        if (val["max-repetitions"]) {
          this.repetitionsMax = val["max-repetitions"];
        }

        if (val.display && val.display.length > 0) {
          this.displayX = val.display[0];
          this.displayY = val.display[1];
        }

        if (val.type === "DIALOGUE") {
          this.textInputs = val.text;
        } else {
          this.textInputs[0] = val.control;
        }
      },
    },
  },
  computed: {
    // returns true if a trigger can be added with current data - i.e. if a button of each type has been selected;
    canAddTrigger() {
      return (
        this.findTrueOptionUppercase(this.typeButtons) &&
        this.findTrueOptionUppercase(this.actionButtons) &&
        this.findTrueOptionUppercase(this.classButtons) &&
        this.findTrueOptionUppercase(this.itemButtons)
      );
    },
  },
  methods: {
    // add a trigger to the running list of triggers on button submit
    addTrigger() {
      let trigger = {
        type: this.findTrueOptionUppercase(this.typeButtons),
        action: this.findTrueOptionUppercase(this.actionButtons),
        "trigger-class": this.findTrueOptionUppercase(this.classButtons),
        position: [Number(this.positionX), Number(this.positionY)],
        dimensions: [Number(this.dimensionsX), Number(this.dimensionsY)],
      };

      // do not set item-type if it is currently "NONE"
      let item = this.findTrueOptionUppercase(this.itemButtons);
      if (item !== "NONE") {
        trigger["item-type"] = item;
      }

      // set repetitions fields only if filled
      if (this.repetitionsMin) {
        trigger["min-repetitions"] = Number(this.repetitionsMin);
      }
      if (this.repetitionsMax) {
        trigger["max-repetitions"] = Number(this.repetitionsMax);
      }

      // only set display if of type control, and set page 1 of text to control property
      if (trigger.type === "CONTROL") {
        if (this.displayX && this.displayY) {
          trigger.display = [Number(this.displayX), Number(this.displayY)];
        } else {
          // set display to an empty array if numbers not provided
          trigger.display = [];
        }
        trigger.control = this.textInputs[0];
      } else {
        // if no elements are empty, add textInputs as is. Otherwise, slice up to first empty element
        let index = this.textInputs.indexOf("");
        if (index < 0) {
          trigger.text = this.textInputs;
        } else {
          trigger.text = this.textInputs.slice(0, index);
        }
      }

      this.$emit("add-trigger", trigger);
      this.clearData();
    },
    // returns the single option in a list of buttons that is set to true in all uppercase
    findTrueOptionUppercase(list) {
      for (const key in list) {
        if (list[key]) {
          return key.toUpperCase();
        }
      }

      return "";
    },
    // set the key equivalent to the option string to true in list, set all others to false
    selectOption(list, option) {
      for (const key in list) {
        if (key === option) {
          list[key] = true;
        } else {
          list[key] = false;
        }
      }

      // special case, but make sure page is set to page 1 if on control
      if (option === "Control") {
        this.currentTextPage = 1;
      }
    },
    // returns the boolean at key option in list
    isSelected(list, option) {
      return list[option];
    },
    // displays the previous page of the text input
    prevPage() {
      if (this.currentTextPage > 1) {
        this.currentTextPage -= 1;
      }
    },
    // displays the next page of the text input. Creates one if does not exist yet
    nextPage() {
      if (this.currentTextPage >= this.textInputs.length) {
        this.textInputs.push("");
      }
      this.currentTextPage += 1;
    },
    // returns true if the pageNum is equal to the current page, and thus should be displayed
    isCurrentPage(pageNum) {
      return pageNum === this.currentTextPage;
    },
    // resets trigger data back to starting point
    clearData() {
      this.resetButtons(this.typeButtons);
      this.resetButtons(this.actionButtons);
      this.resetButtons(this.classButtons);
      this.resetButtons(this.itemButtons);

      this.positionX = 0;
      this.positionY = 0;
      this.dimensionsX = 0;
      this.dimensionsY = 0;
      this.displayX = "";
      this.displayY = "";
      this.repetitionsMin = "";
      this.repetitionsMax = "";
      this.textInputs = [""];
      this.currentTextPage = 1;
    },
    resetButtons(list) {
      for (const key in list) {
        list[key] = false;
      }
    },
    // close the trigger menu
    returnToToolbar() {
      this.$emit("close-triggers");
    },
    // return the string with only the first letter in uppercase
    uppercaseFirstLetter(str) {
      return str.charAt(0).toUpperCase() + str.slice(1).toLowerCase();
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
.sizeInputs {
  margin-top: 0.5rem;
  display: flex;
  justify-content: center;
}

.input {
  max-width: 2rem;
  margin: 0 0.5rem;
}

.repetitionWrapper {
  margin: 0 0.8rem;
  display: flex;
  flex-direction: column;
}

.section {
  margin-top: 1rem;
}
.subSection {
  font-size: 14px;
}

.isDisabled {
  opacity: 0.3;
}

.textArea {
  background-color: #181818;
  color: #88a6cd;
  border-color: #88a6cd;
  margin-top: 0.5rem;
}

.pageText {
  font-size: 12px;
}

.pageButton {
  font-size: 12px;
}

.pageButton:hover {
  color: rgba(16, 16, 16, 0.3);
}

.pageButton:first-child {
  margin-right: 5px;
}

.bottomText {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin: 5px;
}

.bottom {
  margin-top: 1rem;
  display: flex;
  justify-content: space-between;
}

.returnButton {
  background-color: #181818;
  color: #88a6cd;
}

.returnButton:hover {
  background-color: #88a6cd;
  color: black;
}

.inputLabel {
  font-size: 12px;
  text-align: left;
}

.repetitionInput {
  max-width: 2rem;
}
</style>
