<!-- This is the latest version -->

<template>
  <div style="height:1000px; width:1300px">
    <div class="search-overlay">
      <input type="text" class="search-overlay" :placeholder="'From: ' + placeholderText" v-model="searchTerm"
        @input="updateSuggestions(dests)" @blur="hideSuggestions">
      <ul class="search-suggestions" v-if="showSuggestions">
        <li class="search-suggestion" v-for="(suggestion, index) in suggestions" :key="index"
          @mousedown="selectSuggestion(dests, suggestion)">
          {{ suggestion[0] + ", " + suggestion[1] }}
        </li>
      </ul>
    </div>
    <div id="map" style="height:95vh; width:100%"></div>

  </div>
</template>
  
<script>
import "leaflet/dist/leaflet.css";
import * as L from 'leaflet';
import 'leaflet.markercluster';
import 'leaflet.markercluster/dist/MarkerCluster.css';
import 'leaflet.markercluster/dist/MarkerCluster.Default.css'

import westjet from '../assets/westjet.png';
import defaultPopupImage from '../assets/787.jpg';

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
      url: 'https://{s}.basemaps.cartocdn.com/light_nolabels/{z}/{x}/{y}{r}.png',
      attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors &copy; <a href="https://carto.com/attributions">CARTO</a>',
      zoom: 4,
      center: [47.313220, -1.319482], //default
      markerLatLngBob: [47.313220, -17.319482],
      icon: westjet,
      iconSize: [85, 50],
      searchTerm: '',
      suggestions: [],
      suggestion: '',
      showSuggestions: false,
      placeholderText: 'Calgary',
      currentLocationAirportCode: 'YYC',
      noWrap: true,
      searchQuery: '',
      map: null,

    };
  },
  methods: {
    updateSuggestions(dests) {

      var destinations = []
      console.log(destinations)
      for (var index in dests) {
        destinations.push([dests[index].name, dests[index].countryName])
      }
      this.suggestions =
        destinations.filter((suggestion) =>
          suggestion[0].toLowerCase().startsWith(this.searchTerm.toLowerCase())
        );
      // Show the suggestions if there are any.
      this.showSuggestions = this.suggestions.length > 0;
      this.map.closePopup();
    },
    hideSuggestions() {
      // Hide the suggestions when the search input loses focus.
      this.showSuggestions = false;
    },
    selectSuggestion(dests, suggestion) {
      // When a suggestion is clicked, update the search term and hide the suggestions.
      this.searchTerm = "";
      this.showSuggestions = false;
      for (var index in dests) {
        if (dests[index].name === suggestion[0]) {
          this.center = [dests[index].latitude, dests[index].longitude];
          this.placeholderText = dests[index].name;
          this.currentLocationAirportCode = dests[index].code;
          console.log(this.currentLocationAirportCode)
        }
      }

      

    },
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
   
    search(searchQuery, dests) {
      for (var index in dests) {
        var search = searchQuery.toLowerCase()
        var destination = dests[index].name.toLowerCase()
        if (destination.includes(search)) // search centers at the first substring result
        {
          this.center = [dests[index].latitude, dests[index].longitude]
          return;
        }
      }
    },
    openPopup: async function (e) {
      let marker = e.target
      let dest = marker.options.dest


      let from_tag = "From: ";
      let CAD_tag = " CAD"
      //console.log(marker.options.dest.code)
      const price = await this.getPrice(dest);
      const imageUrl = await this.getValidImageUrl(dest.code);

      console.log(price)

      if (price === "This is your current departure location" || price ==="No price available for this route." || price 
      === "This is a seasonal route that is currently not being served."  )
      {
        from_tag = ""
        CAD_tag = ""
      }

      let popupContent = `<h1 style="font-weight: 200;"> ${dest.name} <br>  <div
        style = " font-size: 1rem; font-weight: 200;
        font-family: "Noto Sans","Noto Sans SC","Roboto","Trebuchet MS",Arial,Sans-Serif;
        letter-spacing: normal;
   
   
        line-height: 1.2;"
        >  ${from_tag} ${price} ${CAD_tag} <div> </h1>  `;

      console.log(dest)


      let imageTag = `<img  style="border-radius: 10px;"src=${imageUrl}  width="225" 
       height="225" border-radius: 25px;> </img> <br>`


      var date = new Date();

      // Add five days to current date
      date.setDate(date.getDate() + 30);

      console.log(date);

      var MyDate = new Date();
      var MyDateString;

      MyDate.setDate(MyDate.getDate() + 30);


      MyDateString = MyDate.getFullYear() + '-' + ('0' + (MyDate.getMonth() + 1)).slice(-2) + '-' + ('0' + MyDate.getDate()).slice(-2)

      console.log(MyDateString)


      let button_book = ` https://www.westjet.com/en-ca/flights/low-fares?origin=YQL&destination=${dest.code}&outbounddate=${MyDateString}`


      // let button = ` <div class='text-center'> <button type="button" class="btn btn-outline-dark mt-3 px-5">Book Now!</button> </div>`
      let button = ` <a href=https://www.westjet.com/en-ca/flights/low-fares?origin=${this.currentLocationAirportCode}&destination=${dest.code}&outbounddate=${MyDateString} class="btn btn-outline-dark mt-3 px-5" role="button" ">Book Now!</a>`



      popupContent = popupContent + imageTag + button
      // + '<br>' + "<img src='" + 
      // 'https://images.unsplash.com/photo-1493134799591-2c9eed26201a?ixlib=rb-4.0.3&ixid=MnwxMjA3fDB8MHxzZWFyY2h8Mnx8Y2l0eSUyMHNreWxpbmV8ZW58MHx8MHx8&w=1000&q=80' + "'/>";
      marker.unbindPopup().bindPopup(popupContent,

        { maxWidth: "auto", maxHeight: "auto" },




      ).openPopup();

    },
    getValidImageUrl: async function (dest) {

      let destUrl = `https://www.westjet.com/assets/wj-web/images/en/destination-defaults/square/${dest.toLowerCase()}-square.jpg`

      const axios = require('axios');

      // make a request to the image url to see if it exists
      try {
        const response = await axios.get(destUrl, {});
      } catch(error) {
        //error if 404, so return our default
        return defaultPopupImage;
      }

      return destUrl;

    },
    getPrice: async function (destination) {
      if (destination.code == this.currentLocationAirportCode) {
        return "This is your current departure location";
      }

      let route = this.currentLocationAirportCode + destination.code;
      console.log(route)

      const axios = require('axios');

      let price = "";
      let priceMessage = "";

      const params = new URLSearchParams();
      params.append('o', this.currentLocationAirportCode);
      params.append('d', destination.code);
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
        priceMessage = `$${price}`;
        if (response.data[route][0].status === "NO_SCHEDULES") {

          priceMessage = "This is a seasonal route that is currently not being served."
        }
      } catch (error) {
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
      let marker = L.marker(this.latLng(dest.latitude, dest.longitude), { icon: wjIcon, dest: dest });
      marker.bindTooltip("12", {
        permanent: true,
        direction: 'top',
        offset: [0, -17],
      });
      marker.on('click', this.openPopup)
      marker.bindTooltip(`${dest.name}, ${dest.countryName}`)

      return marker;
    },


    customTip() {
      this.unbindTooltip();
      if (!this.isPopupOpen()) this.bindTooltip('Short description').openTooltip();
    },

    customPop() {
      this.unbindTooltip();
    }

  },
  mounted() {

    this.getLocation();

    var map = L.map("map").setView([51.0447, -114.0719], 4);



    L.tileLayer(
      'https://{s}.basemaps.cartocdn.com/light_nolabels/{z}/{x}/{y}{r}.png',
      {
        minZoom: 4,
        maxZoom: 8,
        attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors &copy; <a href="https://carto.com/attributions">CARTO</a>',


      }
    ).addTo(map);

    let markers = L.markerClusterGroup();

    // create a marker for each airport
    for (let dest of this.dests) {

      let marker = this.createMarker(dest)
      markers.addLayer(marker)

    }

    map.addLayer(markers);
    map.setMaxBounds([[-90, -180], [90, 180]])
    this.map = map;
  }
};
</script>
  
<style>
/* leaflet.css styles*/
.leaflet-container a {
	color: #000000;
}

.leaflet-popup-content-wrapper {
	padding: 0.25px;
	text-align: center;
	border-radius: 12px;
}
.leaflet-popup-content {
	margin: 10px 20px 10px 16px;
	line-height: 1.3;
	font-size: 13px;
	font-size: 1.08333em;
	min-height: 1px;
}
.leaflet-popup-tip-container {
	width: 40px;
	height: 20px;
	position: absolute;
	left: 50%;
	margin-top: -1px;
	margin-left: -15px;
	overflow: hidden;
	pointer-events: none;
}
.leaflet-popup-tip {
	background: rgb(255, 255, 255);
	color: #000000;
	box-shadow: 0 3px 14px rgba(0,0,0,0.4);
}
.leaflet-tooltip {
	font-family: "Noto Sans","Noto Sans SC","Roboto","Trebuchet MS",Arial,Sans-Serif;
	font-weight: 200;
	position: absolute;
	padding: 6px;
	background-color: #01A19E;
	border: 1px solid #01A19E;
	border-radius: 3px;
	color: #ffffff;
	white-space: nowrap;
	-webkit-user-select: none;
	-moz-user-select: none;
	-ms-user-select: none;
	user-select: none;
	pointer-events: none;
	box-shadow: 0 1px 3px rgba(0,0,0,0.4);
}
.leaflet-tooltip-top:before {
	bottom: 0;
	margin-bottom: -12px;
	border-top-color: #01A19E;
}
/* leafletmarkercluster styles*/

.marker-cluster-small {
	background-color: #027E7B;
	}
.marker-cluster-small div {
	background-color: #027E7B;
	}

.marker-cluster-medium {
	background-color: #00203E;
	}
.marker-cluster-medium div {
	background-color: #00203E;
	}
.marker-cluster div {
	width: 30px;
	height: 30px;
	margin-left: 5px;
	margin-top: 5px;

	text-align: center;
	border-radius: 15px;
	font: 12px "Helvetica Neue", Arial, Helvetica, sans-serif;
	color: white
}
/* above here are test changes */

.leaflet-container {
  border-radius: 25px;
  color: #ffffff;
  font-family: Avenir, Helvetica, Arial, sans-serif;
  font-weight: 200;
}

.search-overlay {
  position: absolute;
  top: 4.5rem;
  left: 5rem;
  width: 60%;
  height: 20%;
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 999;
  pointer-events: auto;
}

.search-overlay:hover #map-container {
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
  top: 6.5rem;
  left: 5rem;
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

    
    