<template>
  <div>
    <div id="mapContainer"></div>
  </div>
</template>

<script>
  import "leaflet/dist/leaflet.css";
  import "leaflet-draw/dist/leaflet.draw.css";

  import "leaflet-defaulticon-compatibility/dist/leaflet-defaulticon-compatibility.webpack.css"; // Re-uses images from ~leaflet package

  import * as L from "leaflet";
  import "leaflet-draw";
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
        // this.mapDiv = L.map("mapContainer").setView(this.center, 13);
        this.mapDiv = L.map("mapContainer").fitWorld();
        L.tileLayer(`https://api.mapbox.com/styles/v1/mapbox/streets-v11/tiles/{z}/{x}/{y}?access_token=${this.accessToken}`).addTo(this.mapDiv);
        this.mapDiv.locate({ setView: true, maxZoom: 16 });

        this.mapDiv.on("locationfound", this.onLocationFound);
        this.mapDiv.on("locationerror", this.onLocationError);

        var editableLayers = new L.FeatureGroup();
        this.mapDiv.addLayer(editableLayers);
        var drawnItems = new L.FeatureGroup();
        this.mapDiv.addLayer(drawnItems);

        var drawControl = new L.Control.Draw({
          draw: {
            polyline: {
              shapeOptions: {
                color: "#f357a1",
                weight: 10,
              },
            },
            simpleshape: true,

            polygon: {
              allowIntersection: false, // Restricts shapes to simple polygons
              drawError: {
                color: "#e1e100", // Color the shape will turn when intersects
                message: "<strong>Polygon draw does not allow intersections!<strong> (allowIntersection: false)", // Message that will show when intersect
              },
              shapeOptions: {
                color: "#bada55",
              },
            },
            circle: true, // Turns off this drawing tool
            rectangle: {
              shapeOptions: {
                clickable: false,
              },
            },
          },
          edit: {
            featureGroup: drawnItems,
            remove: false,
          },
        });

        this.mapDiv.addControl(drawControl);

        drawControl.setDrawingOptions({
          rectangle: {
            shapeOptions: {
              color: "#0000FF",
            },
          },
          simpleShape: {
            shapeOptions: {
              color: "#f357a1",
              weight: 10,
            },
          },
        });

        this.mapDiv.on(L.Draw.Event.CREATED, function(e) {
          var type = e.layerType,
            layer = e.layer;

          if (type === "marker") {
            layer.bindPopup("A popup!");
          }

          editableLayers.addLayer(layer);
        });
      },
      onLocationFound(e) {
        var radius = e.accuracy;

        L.marker(e.latlng)
          .addTo(this.mapDiv)
          .bindPopup("You are within " + radius + " meters from this point")
          .openPopup();

        L.circle(e.latlng, radius).addTo(this.mapDiv);
      },
      onLocationError(e) {
        alert(e.message);
        this.mapDiv.setView(this.center, 13);
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
