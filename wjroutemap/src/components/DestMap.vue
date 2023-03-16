<template>
  <div style="height:1000px; width:1300px">
 
 
    <l-map ref="map" :zoom="zoom" :center="center" maxZoom="7" minZoom="4">
      <l-tile-layer
        url="https://tile.thunderforest.com/transport-dark/{z}/{x}/{y}.png?apikey=9b2313ed32304004a51c1494aedf88db"
        layer-type="base" name="OpenStreetMap"></l-tile-layer>
      <l-marker :key="index" v-for="(dest, index) in dests" :lat-lng="latLng(dest.latitude, dest.longitude)">
        <l-icon :icon-size="iconSize" :icon-url="icon"></l-icon>
        <l-popup> You have selected {{ dest.name }}'s airport <br>
         
          Price: {{ getPrice(dest.code) }}
        </l-popup>
        <l-tooltip>
          {{ dest.name }}, {{ dest.countryName }}, {{getPrice(dest.code)}}</l-tooltip>
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
    };
  },
  methods: {
    latLng: function (lat, lng) {
 
 
      return [lat, lng]
    },
    getLocation: function () {
      if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition((position) => {
          var latitude = position.coords.latitude;
          var longitude = position.coords.longitude;
          this.center = [latitude, longitude]
        });
      }
    },
    // dispPrice: async function (cityCode){
    //   const result = await this.getPrice(cityCode);
    //   console.log(result)
    //   return result;
 
    // },
    getPrice: function (destination) {
 
 
      let route = "YYC"+ destination
 
 
      let value = ""
 
 
      const axios = require('axios');
 
 
      const params = new URLSearchParams();
      params.append('o', 'YYC');
      //console.log(destination)
      params.append('d', destination);
      //params.append('sourceCountryCode', 'CA');
      params.append('rangeStartOffset', '0');
      params.append('rangeEndOffset', '60');

     
 
      const response = axios.get('https://api.westjet.com/price-points/v1/retail', {
        params: params,
        headers: {
          'accept': 'application/json'
        }
      }).then(response => {
        value = (response.data[route][0].price);
      }).catch(error => {
        console.log(error)
      });
 
      if (value == 'undefined'){
        value = 'Price Not Available'
      }
      
      //console.log(route + " " + value)
      return value;
    },
  },
  mounted() {
    this.getLocation()

  },
 };
 </script>
  <style>
 .leaflet-container {
  border-radius: 25px;
 }
 </style>
 
 
   
   
 
 