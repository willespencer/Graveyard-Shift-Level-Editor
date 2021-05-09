<template>
  <div class="triggers">
    <h4>Triggers</h4>
    <div class="field">
      <div class="section">Type</div>
      <div class="buttons">
        <button class="button">Dialogue</button>
        <button class="button">Control</button>
      </div>
    </div>
    <div class="field">
      <div class="section">Action</div>
      <div class="buttons">
        <button class="button">Enter</button>
        <button class="button">Pickup</button>
        <button class="button">Use</button>
        <button class="button">UseEnter</button>
        <button class="button">Drop</button>
        <button class="button">DropEnter</button>
      </div>
    </div>
  </div>
</template>

<script>
export default {};
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
</style>
