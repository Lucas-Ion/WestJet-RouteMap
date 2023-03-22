<template>
  <div style="height:600px; width:800px">
    <l-map ref="map" :zoom="zoom" :center="center" maxZoom="7" minZoom="4">
      <div class="search-overlay">
        <input type="text" placeholder="Search" v-model="searchQuery" @keyup.enter="search(searchQuery, dests)">
      </div>
      <l-tile-layer url="https://tile.thunderforest.com/transport-dark/{z}/{x}/{y}.png?apikey=9b2313ed32304004a51c1494aedf88db" layer-type="base" name="OpenStreetMap"></l-tile-layer>
      <l-marker :key="index" v-for="(dest, index) in dests" :lat-lng="latLng(dest.latitude, dest.longitude)">
        <l-icon :icon-size="iconSize" :icon-url="icon"></l-icon>
        <l-popup> You have selected {{dest.name}}'s airport </l-popup>
        <l-tooltip> {{ dest.name }}, {{dest.countryName}} </l-tooltip>
      </l-marker>
    </l-map>
  </div>
</template>
  
<script>
  import "leaflet/dist/leaflet.css";
  import {
    LMap,
    LTileLayer,
    LMarker,
    LIcon,
    LTooltip,
    LPopup,
  } from "@vue-leaflet/vue-leaflet";
  import westjet from '../assets/westjet.png';
  export default {
    name: 'DestMap',
    props: {
      dests: Array
    },
    components: {
      LMap,
      LTileLayer,
      LMarker,
      LIcon,
      LPopup,
      LTooltip,
    },
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
    },
    mounted() {
      this.getLocation()
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

    
    