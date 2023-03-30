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

        let popupContent = `<h1 class ='h6 text-center'> ${dest.name} <br> $${price} </h1> <br>`;
        console.log(dest)
         let imageSource = `<img src="https://www.westjet.com/assets/wj-web/images/en/destination-defaults/square/${dest.code.toLowerCase()}-square.jpg"  width="200" 
     height="200" border-radius: 50%;> </img>`
         console.log(imageSource)
         popupContent = popupContent +  imageSource
        // + '<br>' + "<img src='" + 
        // 'https://images.unsplash.com/photo-1493134799591-2c9eed26201a?ixlib=rb-4.0.3&ixid=MnwxMjA3fDB8MHxzZWFyY2h8Mnx8Y2l0eSUyMHNreWxpbmV8ZW58MHx8MHx8&w=1000&q=80' + "'/>";
        marker.unbindPopup().bindPopup(popupContent,
        
         {maxWidth: "auto", maxHeight: "auto"},
        
        
        
        
        ).openPopup();

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
        //<l-tooltip> {{ dest.name }}, {{dest.countryName}} </l-tooltip>
        let marker = L.marker(this.latLng(dest.latitude, dest.longitude), {icon: wjIcon, dest: dest});
        marker.bindTooltip("12" ,{
        permanent: true, 
        direction: 'top',
        offset: [0,-17],
    });
        marker.on('click', this.openPopup)
        marker.bindTooltip(`${dest.name}, ${dest.countryName}`)
      
        return marker;
      },


      customTip() {
    this.unbindTooltip();
    if(!this.isPopupOpen()) this.bindTooltip('Short description').openTooltip();
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
      for(let dest of this.dests) {
    
        let marker = this.createMarker(dest)
        markers.addLayer(marker)

      }

      map.addLayer(markers);
      map.setMaxBounds(  [[-90,-180],   [90,180]]  )
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

    
    