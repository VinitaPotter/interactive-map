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
        e: null,
        center: [51.505, -0.09],
        mapDiv: null,
        line: "",
        drawing: false,
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
            // circlemarker: {
            //   shapeOptions: {
            //     color: "#f357a1",
            //     weight: 10,
            //   },
            // },
          },
          edit: {
            featureGroup: drawnItems,
            remove: false,
          },
        });

        // var drawControl = new L.Control.Draw();

        this.mapDiv.addControl(drawControl);

        drawControl.setDrawingOptions({
          rectangle: {
            shapeOptions: {
              color: "#0000FF",
            },
          },
          // simpleShape: {
          //   shapeOptions: {
          //     color: "#f357a1",
          //     weight: 10,
          //   },
          // },
        });
        this.mapDiv.on("click", (e) => {
          if (!L.Browser.mobile) {
            this.e = null;
            this.e = e;
            this.freehand_draw();
            // if (this.drawing) {
            //   this.stop_freehand();
            //   // } else {
            // }
          }
        });
        this.mapDiv.on("touchstart", (e) => {
          if (L.Browser.mobile) {
            console.log("touchstart", e);
            // this.e = null;
            this.e = e;
            // this.drawing = !this.drawing;
            this.freehand_draw();
          }
        });
        this.mapDiv.on("touchmove", (e) => {
          if (L.Browser.mobile) {
            this.e = e;
            this.drawLine(e);
            if (this.drawing) {
              this.mapDiv.dragging.disable();
            }
            // console.log("touchmove", e);
            // this.e = null;
            // this.freehand_draw(e);
          }
        });
        // this.mapDiv.on("touchend", (e) => {
        //   console.log("touchend");
        //   if (L.Browser.mobile) {
        //     this.e = null;
        //     this.mapDiv.dragging.enable();
        //     this.stop_freehand();
        //   }
        // });
        // this.mapDiv.on("touchcancel", (e) => {
        //   console.log("touchcancel");
        // });
        this.mapDiv.on(L.Draw.Event.CREATED, (e) => {
          this.e = null;

          var type = e.layerType,
            layer = e.layer;

          if (type === "marker") {
            layer.bindPopup("A popup!");
          }
          if (type === "circlemarker") {
            // this.drawing = true;

            this.freehand_draw(e);
          }

          editableLayers.addLayer(layer);
        });
        // this.mapDiv.on("mousemove", (e) => {
        //   this.throttle(this.drawLine(e), 25);
        // });
        // this.mapDiv.on("mousemove", this.drawLine);
        this.mapDiv.on("mousemove", this.throttle(this.drawLine, 25));
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
      drawLine(e) {
        if (this.drawing) {
          console.log("drawing..");
          this.mapDiv.touchZoom.disable();
          this.mapDiv.doubleClickZoom.disable();
          this.mapDiv.scrollWheelZoom.disable();
          this.mapDiv.boxZoom.disable();
          this.mapDiv.keyboard.disable();
          this.line.addLatLng(e.latlng);
        }
      },
      throttle(func, limit) {
        let inThrottle;
        return function() {
          const args = arguments;
          const context = this;
          if (!inThrottle) {
            func.apply(context, args);
            inThrottle = true;
            setTimeout(() => (inThrottle = false), limit);
          }
        };
      },
      freehand_draw(e) {
        if (!this.drawing) {
          this.drawing = true;
          this.line = L.polyline([]).addTo(this.mapDiv);
          this.line.addLatLng(this.e.latlng);
          console.log("ADDED LINE");
          // this.line.setStyle({
          //   "color": "#fc032c",
          //   "opacity": 1,
          //   "weight": 10,
          // });
        } else {
          this.stop_freehand();
        }
      },
      stop_freehand() {
        this.drawing = false;
        this.mapDiv.dragging.enable();
        this.mapDiv.touchZoom.enable();
        this.mapDiv.doubleClickZoom.enable();
        this.mapDiv.scrollWheelZoom.enable();
        this.mapDiv.boxZoom.enable();
        this.mapDiv.keyboard.enable();
        console.log("Closed line", this.line);
      },
    },
  };
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
  #mapContainer {
    width: 100vw;
    height: 80vh;
    cursor: url("~@/assets/pen.svg"), crosshair !important;
    @media only screen and (max-width: 600px) {
      body {
        height: 100vh;
      }
    }
  }
</style>
