<template>
  <div id="app">
    <Dashboard />
  </div>
</template>

<script>
import Dashboard from "./components/Dashboard.vue";

export default {
  name: "App",
  components: {
    Dashboard,
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #88a6cd;
  background-color: #181818;
  min-height: 100vh;
}

body {
  margin: 0;
}

button {
  background-color: #88a6cd;
  border: 1px solid #88a6cd;
  border-radius: 2px;
  cursor: pointer;
}

button:hover:not([disabled]) {
  background-color: #51616e;
  border-color: #51616e;
}

button:disabled {
  background-color: #51616e;
  border-color: #51616e;
  cursor: auto;
}

input {
  background-color: #181818;
  color: #88a6cd;
  border: 1px solid #88a6cd;
}
</style>
