<template>
  <div style="height:1000px; width:1300px">
    <l-map ref="map" :zoom="zoom" :center="center" :maxZoom=7 :minZoom=4>
      <div class="search-overlay">
        <input type="text" class="search-overlay" placeholder="Search" v-model="searchTerm" @input="updateSuggestions(dests)" @blur="hideSuggestions">
        <ul class="search-suggestions" v-if="showSuggestions">
          <li class="search-suggestion" v-for="(suggestion, index) in suggestions" :key="index" @mousedown="selectSuggestion(dests, suggestion)">
            {{ suggestion[0] + ", " + suggestion[1] }}
          </li>
        </ul>
      </div>
      <l-tile-layer url="https://tile.thunderforest.com/transport-dark/{z}/{x}/{y}.png?apikey=9b2313ed32304004a51c1494aedf88db" layer-type="base" name="OpenStreetMap"></l-tile-layer>
      <l-marker ref="markers" :key="index" v-for="(dest, index) in dests" :lat-lng="latLng(dest.latitude, dest.longitude)" @click="openPopup(dest, index)">
        <l-icon :icon-size="iconSize" :icon-url="icon"></l-icon>
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
    LTooltip
  } from "@vue-leaflet/vue-leaflet";
  import westjet from '../assets/westjet.png';
import { map } from "leaflet";
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
        searchTerm: '',
        suggestions: [],
        suggestion: '',
        showSuggestions: false,
      };
    },
    methods: {
      updateSuggestions(dests) {
        var destinations = []
        for (var index in dests) {
          destinations.push([dests[index].name, dests[index].countryName])
        }
        this.suggestions =
          destinations.filter((suggestion) =>
            suggestion[0].toLowerCase().startsWith(this.searchTerm.toLowerCase())
          );
        // Show the suggestions if there are any.
        this.showSuggestions = this.suggestions.length > 0;
      },
      hideSuggestions() {
        // Hide the suggestions when the search input loses focus.
        this.showSuggestions = false;
      },
      selectSuggestion(dests, suggestion) {
        // When a suggestion is clicked, update the search term and hide the suggestions.
        this.searchTerm = suggestion[0];
        this.showSuggestions = false;
        for (var index in dests) {
          if (dests[index].name === suggestion[0]) {
            this.center = [dests[index].latitude, dests[index].longitude]
          }
        }
      },
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
      openPopup: async function(dest, index) {
        let marker = this.$refs.markers[index].leafletObject;
        const price = await this.getPrice(dest.code);
        let popupContent = `You have selected ${dest.name}'s airport <br> Price: ${price}`;
        marker.unbindPopup().bindPopup(popupContent).openPopup();
      },
      getPrice: async function(destination) {
        let route = "YYC" + destination
        const axios = require('axios');
        let price = ""
        const params = new URLSearchParams();
        params.append('o', 'YYC');
        params.append('d', destination);
        params.append('rangeStartOffset', '0');
        params.append('rangeEndOffset', '60');
        try {
          const response = await axios.get('https://api.westjet.com/price-points/v1/retail', {
            params: params,
            headers: {
              'accept': 'application/json'
            }
          });
          price = response.data[route][0].price;
        } catch (error) {
          console.log(error);
          price = "No price available.";
        }
        return price;
      }
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
    top: 10px;
    left: 30px;
    width: 60%;
    height: 20%;
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 999;
    pointer-events: auto;
  }
  .search-overlay:hover #map-container{
    pointer-events: none;
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
  .search-container {
    display: flex;
    justify-content: center;
    margin: 20px 0;
  }
  .search-bar {
    position: relative;
  }
  .search-input {
    width: 300px;
    padding: 8px;
    border: none;
    border-radius: 5px;
  }
  .search-suggestions {
    position: absolute;
    top: 45px;
    left: 30px;
    z-index: 1;
    width: 300px;
    margin: 0;
    padding: 0;
    list-style: none;
    background-color: #fff;
    border: 1px solid #ccc;
    border-radius: 5px;
  }
  .search-suggestion {
    padding: 8px;
    cursor: pointer;
  }
  .search-suggestion:hover {
    background-color: #f2f2f2;
  }
</style>

    
    