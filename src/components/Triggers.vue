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
      <div class="section">Text</div>
      <div v-for="index in textInputs.length" :key="index">
        <div v-if="isCurrentPage(index)">
          <textarea
            class="textArea"
            v-model="textInputs[index - 1]"
            rows="7"
          ></textarea>
          <div class="bottomText">
            <span class="pageText">Page {{ index }}</span>
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
      positionX: 0,
      positionY: 0,
      dimensionsX: 0,
      dimensionsY: 0,
      displayX: 0,
      displayY: 0,
      textInputs: [""],
      currentTextPage: 1,
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
</style>
