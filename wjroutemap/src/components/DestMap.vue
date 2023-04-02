<template>
  <div style="height:1000px; width:1300px">
    <div id="map" style="height:1000px; width:1300px"></div>

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
      url: 'https://{s}.basemaps.cartocdn.com/light_nolabels/{z}/{x}/{y}{r}.png',
      attribution: '&copy; <a target="_blank" href="http://osm.org/copyright">OpenStreetMap</a> contributors',
      zoom: 4,
      center: [47.313220, -1.319482], //default
      markerLatLngBob: [47.313220, -17.319482],
      icon: westjet,
      iconSize: [85, 50],
      searchQuery: '',
      noWrap: true,

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
      //console.log(marker.options.dest.code)
      const price = await this.getPrice(dest.code);
      const valid_image = await this.getValidImage(dest.code);

      let popupContent = `<h1 style="font-weight: 200;"> ${dest.name} <br>  <div
        style = " font-size: 1rem; font-weight: 200;
    font-family: "Noto Sans","Noto Sans SC","Roboto","Trebuchet MS",Arial,Sans-Serif;
    letter-spacing: normal;
   
   
    line-height: 1.2;"
    >  From $${price} CAD <div> </h1>  `;
      console.log(dest)





      let imageSource = `<img  style="border-radius: 10px;"src="https://www.westjet.com/assets/wj-web/images/en/destination-defaults/square/${dest.code.toLowerCase()}-square.jpg"  width="225" 
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
      let button = ` <a href=https://www.westjet.com/en-ca/flights/low-fares?origin=YQL&destination=${dest.code}&outbounddate=${MyDateString} class="btn btn-outline-dark mt-3 px-5" role="button" ">Book Now!</a>`



      popupContent = popupContent + imageSource + button
      // + '<br>' + "<img src='" + 
      // 'https://images.unsplash.com/photo-1493134799591-2c9eed26201a?ixlib=rb-4.0.3&ixid=MnwxMjA3fDB8MHxzZWFyY2h8Mnx8Y2l0eSUyMHNreWxpbmV8ZW58MHx8MHx8&w=1000&q=80' + "'/>";
      marker.unbindPopup().bindPopup(popupContent,

        { maxWidth: "auto", maxHeight: "auto" },




      ).openPopup();

    },
    getValidImage: function (dest) {






    },
    getPrice: async function (destination) {

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
        if (response.data[route][0].status === "NO_SCHEDULES") {
          console.log("h")
          price = "This is a seasonal route that is currently not being served."
        }
      } catch (error) {
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

    var map = L.map("map").setView([47.313220, -1.319482], 4);



    L.tileLayer(
      'https://{s}.basemaps.cartocdn.com/light_nolabels/{z}/{x}/{y}{r}.png',
      {
        minZoom: 4,
        maxZoom: 8,
        attribution: 'Maps <a href=&copy; https://www.thunderforest.com/>Thunderforest</a>, Data &copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap Contributors</a>',


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
  }
};
</script>
  
<style>
.leaflet-container {
  border-radius: 25px;
  color: #ffffff;
  font-family: Avenir, Helvetica, Arial, sans-serif;
  font-weight: 200;

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

    
    