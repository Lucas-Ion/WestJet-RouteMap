<template>
  <div class="container">
    <div class="row">
      <div class="col-12">
        <h1 class="text-center text-info">WestJet Route Finder</h1>
      </div>
      <!-- col -->
    </div>
    <!-- row -->
    <div class="row">
      <div class="col-6">
        <h1 class="mt-5 mb-5">List</h1>
        <DestList :dests="dests" />
        <!-- DestList Component -->
      </div>
      <!-- left column -->
      <div class="col-6">
        <h1 class="mt-5 mb-5">Map</h1>
        <DestMap :dests="dests" />
        <!-- Insert Map Here -->
      </div>
      <!-- right column -->
    </div>
    <!-- row -->
  </div>
  <!-- container -->
</template>

<script>
  import DestList from './DestList.vue';
  import DestMap from './DestMap.vue';
  /* eslint-disable */
  import axios from 'axios';
  // https://api.westjet.com/destination-service/v1/en-CA/origins/YYC/destinations/westjet-only
  export default {
    name: 'RouteMap',
    data: function() {
      return {
        dests: []
      }
    },
    components: {
      DestList,
      DestMap,
    },
    mounted: function() {
      console.log('mounted');
      axios.get('https://api.westjet.com/destination-service/v1/en-CA/airports')
        .then((r) => {
          console.log(r.data.airports)
          this.dests = r.data.airports
        })
    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped>

</style>
