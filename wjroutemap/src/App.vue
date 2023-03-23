<!-- <template>
  <div class="container-fluid">
    <RouteMap/>
  </div>
</template>

<script>
  import RouteMap from './components/RouteMap.vue'
  export default {
    name: 'App',
    components: {
      RouteMap
    }
  }
</script>

<style>
  #app {
    font-family: Avenir, Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin-top: 60px;
  }
</style> -->
<template>
  <div class="map-test">
    <test-map />
  </div>
</template>

<script>
import TestMap from "./components/TestMap.vue";

export default {
  components: {
    TestMap,
  },
};
</script>

<style>
.map-test {
  height: 500px;
}
</style>

