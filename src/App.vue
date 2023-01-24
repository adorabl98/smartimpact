<template>
  <div class="app">
    <Homepage />
  </div>
</template>
<script>
import Homepage from "./pages/Home.vue";

export default {
  name: "App",
  components: {
    Homepage,
  },
};
</script>

<style>
* {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  margin: 0;
  padding: 0;
}

.app {
  margin-inline: 2rem;
}
</style>
