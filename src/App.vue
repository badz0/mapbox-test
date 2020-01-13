<template>
  <div id="app">
    <Mapbox />
  </div>
</template>

<script>
import Mapbox from './components/Mapbox.vue';

export default {
  name: 'app',
  components: {
    Mapbox,
  },
};
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
}
body, p { margin: 0; padding: 0; }
#map { position: absolute; top: 0; bottom: 0; width: 100%; };
</style>
