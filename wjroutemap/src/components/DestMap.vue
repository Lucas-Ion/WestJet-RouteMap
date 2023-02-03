<template>
  <div style="height:600px; width:800px">
    <l-map ref="map" v-model:zoom="zoom" :center="[50, -100]" maxZoom=7 minZoom=2>
      <l-tile-layer url="https://tile.thunderforest.com/transport-dark/{z}/{x}/{y}.png?apikey=9b2313ed32304004a51c1494aedf88db" layer-type="base" name="OpenStreetMap"></l-tile-layer>
      <l-marker v-for="(dest, index) in dests" :key="index" 
          :lat-lng="latLng(dest.latitude, dest.longitude)">
        <l-icon :icon-size="iconSize" :icon-url="icon"></l-icon>
        <l-tooltip> 
          {{ dest.name }}, {{dest.countryName}}</l-tooltip>
        <l-popup>You have selecter {{ dest.name }}'s popup.</l-popup>
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
  import westjet from '../assets/westjet.png'
  
  export default {
    name: 'DestMap',
    props:{
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
          attribution:
            '&copy; <a target="_blank" href="http://osm.org/copyright">OpenStreetMap</a> contributors',
          zoom: 3,
          center: [47.313220, -1.319482],
          markerLatLngBob: [47.313220, -17.319482],
          icon: westjet,
          iconSize: [75,50]
      };
    },
    methods: {
        latLng: function(lat, lng){
            return[lat,lng]
        },
    }
    
  };
  </script>
  
  <style>
  .leaflet-container{
    border-radius: 25px;
    }</style>

    
    