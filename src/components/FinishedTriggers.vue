<template>
  <div class="finished">
    <h3>Finished Triggers</h3>
    <section class="section">
      <header class="header">
        <span class="col"></span>
        <span class="col">Type</span>
        <span class="col">Pos</span>
        <span class="col">Dim</span>
        <span class="col">Action</span>
        <span class="col">Trigger</span>
        <span class="col">Item</span>
        <span class="col">Display</span>
        <span class="col">Min R</span>
        <span class="col">Max R</span>
        <span class="col">Text / Control</span>
      </header>
      <div class="row" v-for="index in triggerList.length" :key="index">
        <span class="col">
          <button class="unstyledButton" @click="deleteTrigger(index - 1)">
            <img class="trash" src="@/assets/trash.svg" />
          </button>
          <button class="unstyledButton" @click="editTrigger(index - 1)">
            <img class="edit" src="@/assets/edit.svg" />
          </button>
        </span>
        <span class="col">{{ triggerList[index - 1].type }}</span>
        <span class="col">{{
          getArrayString(triggerList[index - 1].position)
        }}</span>
        <span class="col">{{
          getArrayString(triggerList[index - 1].dimensions)
        }}</span>
        <span class="col">{{ triggerList[index - 1].action }}</span>
        <span class="col">{{ getClass(index - 1) }}</span>
        <span class="col">{{ getItem(index - 1) }}</span>
        <span class="col">{{ getDisplay(index - 1) }}</span>
        <span class="col">{{ getMinReps(index - 1) }}</span>
        <span class="col">{{ getMaxReps(index - 1) }}</span>
        <span class="col">{{ getText(index - 1) }}</span>
      </div>
    </section>
  </div>
</template>

<script>
export default {
  data() {
    return {};
  },
  props: {
    triggerList: Array,
  },
  methods: {
    deleteTrigger(index) {
      this.$emit("delete-trigger", index);
    },
    editTrigger(index) {
      this.$emit("edit-trigger", index);
    },
    getText(index) {
      if (this.triggerList[index].type === "DIALOGUE") {
        return this.triggerList[index].text;
      } else {
        return this.triggerList[index].control;
      }
    },
    getClass(index) {
      let triggerClass = this.triggerList[index]["trigger-class"];
      if (triggerClass === "") {
        return "PLAYER";
      }
      return triggerClass;
    },
    getItem(index) {
      let triggerItem = this.triggerList[index]["item-type"];
      if (!triggerItem || triggerItem === "") {
        return "";
      }
      return triggerItem;
    },
    getDisplay(index) {
      if (this.triggerList[index].display) {
        return this.getArrayString(this.triggerList[index].display);
      } else {
        return "";
      }
    },
    getMinReps(index) {
      if (this.triggerList[index]["min-repetitions"]) {
        return this.triggerList[index]["min-repetitions"];
      }

      return "";
    },
    getMaxReps(index) {
      if (this.triggerList[index]["max-repetitions"]) {
        return this.triggerList[index]["max-repetitions"];
      }

      return "";
    },
    getArrayString(arr) {
      if (arr.length === 0) {
        return "[ ]";
      }
      if (arr.length === 1) {
        return `[${arr[0]}]`;
      }
      return `[${arr[0]}, ${arr[1]}]`;
    },
  },
};
</script>

<style scoped>
.section {
  display: table;
  width: 100%;
}

.row,
.header {
  display: table-row;
}

.col {
  display: table-cell;
  border: 1px dashed #51616e;
  padding: 2px;
  vertical-align: middle;
  min-width: 60px;
}

.trash {
  height: 15px;
  color: red;
}

.edit {
  margin-left: 3px;
  height: 15px;
}

.unstyledButton {
  padding: 0;
  border: none;
  border-radius: 0;
  background-color: transparent;
}

.unstyledButton:hover {
  background-color: transparent;
}

.unstyledButton:first-child {
  margin-right: 3px;
}

.unstyledButton:last-child {
  margin-left: 3px;
}
</style>
