<template>
    <l-map
      ref="map"
      :max-zoom="19"
      :zoom="zoom"
      :center="{ lat: 51.289404225298256, lng: 9.697202050919614 }"
      @ready="onLeafletReady"
    >
      <template v-if="leafletReady">
        <l-tile-layer :url="url" />
        <marker-cluster
          :options="{ showCoverageOnHover: false, chunkedLoading: true }"
        >
          <l-marker :lat-lng="[47.7515953048815, 8.757179159967961]" />
          <l-marker :lat-lng="[54.379448751829784, 8.890621239746661]" />
          <l-marker :lat-lng="[48.41432462648719, 11.172363685423019]" />
          <l-marker :lat-lng="[54.34757868763789, 11.410597389004957]" />
          <l-marker :lat-lng="[51.741295879474464, 13.693138753473695]" />
          <l-marker :lat-lng="[53.574845165295145, 6.875185458821902]" />
          <l-marker :lat-lng="[51.42494690949777, 6.901031944520698]" />
        </marker-cluster>
      </template>
    </l-map>
  </template>
  <script>
  import "leaflet/dist/leaflet.css";
  
  import { LMap, LMarker, LTileLayer } from "@vue-leaflet/vue-leaflet/src/lib";
  import {MarkerCluster} from "./MarkerCluster.vue";
  
  export default {
    components: {
      LMap,
      LTileLayer,
      LMarker,
      MarkerCluster,
    },
  
    data() {
      return {
        url: "https://{s}.tile.osm.org/{z}/{x}/{y}.png",
        zoom: 1,
  
        leafletReady: false,
        leafletObject: null,
  
        visible: false,
      };
    },
  
    methods: {
      async onLeafletReady() {
        await this.$nextTick();
        this.leafletObject = this.$refs.map.leafletObject;
        this.leafletReady = true;
      },
    },
  };
  </script>
  