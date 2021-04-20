<template>
  <div>
    <div id="mapContainer"></div>
  </div>
</template>

<script>
  import "leaflet/dist/leaflet.css";
  import "leaflet-defaulticon-compatibility/dist/leaflet-defaulticon-compatibility.webpack.css"; // Re-uses images from ~leaflet package
  import * as L from "leaflet";
  import "leaflet-defaulticon-compatibility";

  export default {
    name: "Map",
    data() {
      return {
        center: [51.505, -0.09],
        mapDiv: null,
        accessToken: "pk.eyJ1IjoidGVzdGVyMjM4Nzg2MjciLCJhIjoiY2tub2k2OWFrMHlxcDJ2bzVnbm5na213cyJ9.gpHnP345S2_zJHsj_Zx46A",
      };
    },
    mounted() {
      this.setupLeafletMap();
    },
    methods: {
      setupLeafletMap: function() {
        this.mapDiv = L.map("mapContainer").setView(this.center, 13);
        L.tileLayer(`https://api.mapbox.com/styles/v1/mapbox/streets-v11/tiles/{z}/{x}/{y}?access_token=${this.accessToken}`).addTo(this.mapDiv);

        var marker = L.marker([51.5, -0.09]).addTo(this.mapDiv);
        var polygon = L.polygon([
          [51.509, -0.08],
          [51.503, -0.06],
          [51.51, -0.047],
        ]).addTo(this.mapDiv);
        this.mapDiv.on("click", this.onMapClick);
      },

      onMapClick(e) {
        var popup = L.popup();
        popup
          .setLatLng(e.latlng)
          .setContent("You clicked the map at " + e.latlng.toString())
          .openOn(this.mapDiv);
      },
    },
  };
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
  #mapContainer {
    width: 100vw;
    height: 80vh;
    @media only screen and (max-width: 600px) {
      body {
        height: 100vh;
      }
    }
  }
</style>
