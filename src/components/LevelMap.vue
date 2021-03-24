<template>
  <div class="levelmap">
    <div class="row" v-for="(row, r) in height" v-bind:key="r">
      <div class="col" v-for="(col, c) in width" v-bind:key="c"></div>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    height: Number,
    width: Number,
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
  border: 1px solid black;
  height: 50px;
  width: 50px;
}
</style>
