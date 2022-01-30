<template>
  <div id="app" class="container">
    <MapContainer />
  </div>
</template>

<script>
import MapContainer from "./components/MapContainer.vue";

export default {
  name: "App",
  components: {
    MapContainer,
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
