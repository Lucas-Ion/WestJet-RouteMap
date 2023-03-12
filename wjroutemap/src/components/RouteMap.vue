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
<<<<<<< Updated upstream
      <div class="col-6">
        <h1 class="mt-5 mb-5">List</h1>
        <DestList :dests="dests" />
        <!-- DestList Component -->

      </div>
      <!-- left column -->
      <div class="col-6">

=======

      <!-- left column -->
      <div class="ml-5 col-12">
>>>>>>> Stashed changes
        <h1 class="mt-5 mb-5">Map</h1>
        <DestMap :dests="dests"/>
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
<<<<<<< Updated upstream

=======
>>>>>>> Stashed changes
// https://api.westjet.com/destination-service/v1/en-CA/origins/YYC/destinations/westjet-only
export default {
  name: 'RouteMap',
  data: function () {
    return {
      dests: []
    }
  },
  components: {
    DestList,
    DestMap,
  },
<<<<<<< Updated upstream

=======
>>>>>>> Stashed changes
  mounted: function () {
    console.log('mounted');
    axios.get('https://api.westjet.com/destination-service/v1/en-CA/airports')
      .then((r) => {
        console.log(r.data.airports)
        this.dests = r.data.airports
      })
<<<<<<< Updated upstream
      axios.get('https://api.westjet.com/destination-service/v1/en-CA/airports')
      .then((r) => {
        console.log(r.data.airports)
       
      })
  }
=======
  }
//   mounted: function () {
//     // const axios = require('axios');

//     // const params = new URLSearchParams();
//     // params.append('o', 'YEG');
//     // params.append('d', 'YYC');
//     // params.append('d', 'YYZ');
//     // params.append('sourceCountryCode', 'CA');
//     // params.append('rangeStartOffset', '30');
//     // params.append('rangeEndOffset', '60');
//     // params.append('iwe', '1');
//     // params.append('iwe', '2');
//     // params.append('iwe', '3');
//     // params.append('iwe', '4');
//     // params.append('iwe', '5');

//     // const response = axios.get('https://api.westjet.com/price-points/v1/retail', {
//     //   params: params,
//     //   headers: {
//     //     'accept': 'application/json'
//     //   }
//     // });

    
//   },

>>>>>>> Stashed changes
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped></style>



