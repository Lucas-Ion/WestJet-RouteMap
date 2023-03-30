<template>
  <div class="container">
    <div class="row">
 
 
      <div class="col-12">
        <nav class="navbar navbar-light bg-light mt-0 mb-5">
          <a class="navbar-brand" href="https://www.westjet.com/en-ca">
            <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/f/ff/WestJet_Logo.svg/2560px-WestJet_Logo.svg.png" width="225" height="30" class="d-inline-block align-top mx-3" alt="">
           <span class="mt-5">Route Map</span>
          </a>
        </nav>
 
 
      </div>
      <!-- col -->
 
 
    </div>
    <!-- row -->
    <div class="row">
 
 
      <!-- left column -->
      <div class="ml-5 col-12">
        <DestMap v-if="destsLoaded" :dests="dests"/>
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
  data: function () {
    return {
      destsLoaded: false,
      dests: []
    }
  },
  components: {
    DestList,
    DestMap,
  },
  mounted: function () {
    console.log('mounted');
    axios.get('https://api.westjet.com/destination-service/v1/en-CA/airports')
      .then((r) => {
        console.log("parentMounted")
        this.dests = r.data.airports
        this.destsLoaded = true;
      })
  }
 
 }
 </script>
 
 
 <!-- Add "scoped" attribute to limit CSS to this component only -->
 <style lang="scss" scoped></style>
