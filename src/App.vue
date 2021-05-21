<template>
  <VodPlayer />
</template>

<script >
import VodPlayer from "./components/VodPlayer.vue";

export default {
  name: "Home",
  components: {
    VodPlayer,
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
