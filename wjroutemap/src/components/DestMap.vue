<template>
  <div style="height:1000px; width:1300px">
    <div id="map" style="height:1000px; width:1300px"></div>
    <!-- <l-map ref="map" :zoom="zoom" :center="center" maxZoom="7" minZoom="4">
      <div class="search-overlay">
        <input type="text" placeholder="Search" v-model="searchQuery" @keyup.enter="search(searchQuery, dests)">
      </div>
      <l-tile-layer url="https://tile.thunderforest.com/transport-dark/{z}/{x}/{y}.png?apikey=9b2313ed32304004a51c1494aedf88db" layer-type="base" name="OpenStreetMap"></l-tile-layer>
      <l-marker ref="markers" :key="index" v-for="(dest, index) in dests" :lat-lng="latLng(dest.latitude, dest.longitude)" @click="openPopup(dest, index)">
        <l-icon :icon-size="iconSize" :icon-url="icon"></l-icon>
        <l-tooltip> {{ dest.name }}, {{dest.countryName}} </l-tooltip>
      </l-marker>
    </l-map> -->
  </div>
</template>
  
<script>
  import "leaflet/dist/leaflet.css";
  import * as L from 'leaflet';
  import 'leaflet.markercluster';
  import 'leaflet.markercluster/dist/MarkerCluster.css';
  import 'leaflet.markercluster/dist/MarkerCluster.Default.css'

  import westjet from '../assets/westjet.png';
  export default {
    name: 'DestMap',
    props: {
      dests: Array
    },
    // components: {
    //   LMap,
    //   LTileLayer,
    //   LMarker,
    //   LIcon,
    //   LTooltip,
    // },
    data() {
      return {
        url: 'https://tile.thunderforest.com/atlas/{z}/{x}/{y}.png?apikey=9b2313ed32304004a51c1494aedf88db',
        attribution: '&copy; <a target="_blank" href="http://osm.org/copyright">OpenStreetMap</a> contributors',
        zoom: 4,
        center: [47.313220, -1.319482], //default
        markerLatLngBob: [47.313220, -17.319482],
        icon: westjet,
        iconSize: [40, 25],
        searchQuery: '',
        
      };
    },
    methods: {
      latLng: function(lat, lng) {
        return [lat, lng]
      },
      getLocation: function() {
        if (navigator.geolocation) {
          navigator.geolocation.getCurrentPosition((position) => {
            var latitude = position.coords.latitude;
            var longitude = position.coords.longitude;
            this.center = [latitude, longitude]
          });
        }
      },
      search(searchQuery, dests) {
        for(var index in dests)
        {
          var search = searchQuery.toLowerCase()
          var destination = dests[index].name.toLowerCase()
          if(destination.includes(search)) // search centers at the first substring result
          {
            this.center = [dests[index].latitude, dests[index].longitude]
            return;
          }
        }
      },
      openPopup: async function(e) {
        let marker = e.target
        let dest = marker.options.dest
        //console.log(marker.options.dest.code)
        const price = await this.getPrice(dest.code);

        let popupContent = `You have selected ${dest.name}'s airport <br> Price: $${price}`;
        marker.unbindPopup().bindPopup(popupContent).openPopup();

      },
      getPrice: async function (destination) {
 
        let route = "YYC"+ destination

        const axios = require('axios');

        let price = ""

        const params = new URLSearchParams();
        params.append('o', 'YYC');
        params.append('d', destination);
        params.append('rangeStartOffset', '0');
        params.append('rangeEndOffset', '60');

        try{
          const response = await axios.get('https://api.westjet.com/price-points/v1/retail', {
            params: params,
            headers: {
              'accept': 'application/json'
            }
          });
          price = response.data[route][0].price;
          if(response.data[route][0].status === "NO_SCHEDULES"){
            console.log("h")
            price = "This is a seasonal route that is currently not being served."
          }
        } catch(error) {
          console.log(error);
          price = "No price available.";
        }
        console.log(price)
        return price;
      },
      createMarker(dest) {
        let wjIcon = L.icon({
          iconUrl: this.icon,
          iconSize: this.iconSize
        });

        let marker = L.marker(this.latLng(dest.latitude, dest.longitude), {icon: wjIcon, dest: dest});
        marker.on('click', this.openPopup)
        return marker;
      }
    },
    mounted() {

      this.getLocation();
      //console.log(this.dests)
      var map = L.map("map").setView([47.313220, -1.319482], 4);

      L.tileLayer(
        "https://tile.thunderforest.com/transport-dark/{z}/{x}/{y}.png?apikey=9b2313ed32304004a51c1494aedf88db",
        {
          minZoom: 4,
          maxZoom: 7,
          attribution: 'Maps <a href=&copy; https://www.thunderforest.com/>Thunderforest</a>, Data &copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap Contributors</a>'
        }
      ).addTo(map);
      
      let markers = L.markerClusterGroup();
      
      // create a marker for each airport
      for(let dest of this.dests) {
    
        let marker = this.createMarker(dest)
        markers.addLayer(marker)

      }

      map.addLayer(markers);
    }
  };
</script>
  
<style>
  .leaflet-container {
    border-radius: 25px;
  }

  .search-overlay {
    position: absolute;
    top: 0;
    left: -100px;
    width: 75%;
    height: 12%;
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 999;
  }

  input[type="text"] {
    padding: 10px;
    border-radius: 5px;
    border: none;
    outline: none;
    font-size: 16px;
    background-color: white;
    width: 50%;
  }

</style>

    
    