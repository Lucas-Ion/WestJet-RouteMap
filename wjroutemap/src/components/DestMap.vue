<template>
  <div style="height:1000px; width:1300px">
    <div class="search-overlay">
      <input type="text" class="search-overlay" :placeholder="'From: ' + placeholderText" v-model="searchTerm" @input="updateSuggestions(dests)" @blur="hideSuggestions">
      <ul class="search-suggestions" v-if="showSuggestions">
        <li class="search-suggestion" v-for="(suggestion, index) in suggestions" :key="index" @mousedown="selectSuggestion(dests, suggestion)">
          {{ suggestion[0] + ", " + suggestion[1] }}
        </li>
      </ul>
    </div>
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
  import { conditionalExpression } from "@babel/types";

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
        map: null,
        url: 'https://tile.thunderforest.com/atlas/{z}/{x}/{y}.png?apikey=9b2313ed32304004a51c1494aedf88db',
        attribution: 'Maps <a href=&copy; https://www.thunderforest.com/>Thunderforest</a>, Data &copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap Contributors</a>',
        minZoom: 4,
        maxZoom: 7,
        center: [47.313220, -1.319482], //default
        markerLatLngBob: [47.313220, -17.319482],
        icon: westjet,
        iconSize: [40, 25],
        searchTerm: '',
        suggestions: [],
        suggestion: '',
        showSuggestions: false,
        placeholderText: 'Location Not Selected',
        currentLocationAirportCode: 'YYC'
      };
    },
    watch: {
    // whenever center changes, this function will run
    center(newCenter, oldCenter) {
      this.map.setView(newCenter, this.maxZoom);
    }
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
            this.center = [dests[index].latitude, dests[index].longitude];
            this.placeholderText = dests[index].name;
            this.currentLocationAirportCode = dests[index].code;
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
            this.center = [latitude, longitude];
          });
        }
      },
      openPopup: async function(e) {
        let marker = e.target
        let dest = marker.options.dest
        //console.log(marker.options.dest.code)
        const priceMessage = await this.getPrice(dest);

        // let popupContent = `You have selected ${dest.name}'s airport <br> Price: $${price}`;
        marker.unbindPopup().bindPopup(priceMessage).openPopup();

      },
      getPrice: async function (destination) {
        if(destination.code == this.currentLocationAirportCode)
        {
          return "This is your current departure location";
        }

        let route = this.currentLocationAirportCode + destination.code;

        const axios = require('axios');

        let price = "";
        let priceMessage = "";

        const params = new URLSearchParams();
        params.append('o', this.currentLocationAirportCode);
        params.append('d', destination.code);
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
          priceMessage = `You have selected ${destination.name}'s airport <br> Price: $${price}`;
          if(response.data[route][0].status === "NO_SCHEDULES"){
            
            priceMessage = "This is a seasonal route that is currently not being served."
          }
        } catch(error) {
          console.log(error);
          priceMessage = "No price available for this route.";
        }
        console.log(price)
        return priceMessage;
      },
      createMarker(dest) {
        let wjIcon = L.icon({
          iconUrl: this.icon,
          iconSize: this.iconSize
        });
        //<l-tooltip> {{ dest.name }}, {{dest.countryName}} </l-tooltip>
        let marker = L.marker(this.latLng(dest.latitude, dest.longitude), {icon: wjIcon, dest: dest});
        marker.on('click', this.openPopup)
        marker.bindTooltip(`${dest.name}, ${dest.countryName}`)
        return marker;
      }
    },
    mounted() {
      
      this.map = L.map("map").setView(this.center, 4);

      L.tileLayer(
        this.url,
        {
          minZoom: this.minZoom,
          maxZoom: this.maxZoom,
          attribution: this.attribution
        }
      ).addTo(this.map);
      
      let markers = L.markerClusterGroup();

      // create a marker for each airport
      for(let dest of this.dests) {
    
        let marker = this.createMarker(dest)
        markers.addLayer(marker)

      }

      this.map.addLayer(markers);

      this.getLocation();
    }
  };
</script>
  
<style>
  .leaflet-container {
    border-radius: 25px;
  }
  .search-overlay {
    position: absolute;
    top: 140px;
    left: 195px;
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
    top: 180px;
    left: 200px;
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

    
    