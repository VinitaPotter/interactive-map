<template>
  <div>
    <div id="mapContainer"></div>
    <input
      type="text"
      class="input"
      v-if="prompt"
      :style="{ 'top': e.containerPoint.y + 'px', 'left': e.containerPoint.x + 'px' }"
      @keyup.enter="add_text_to_map"
      v-model="text"
    />
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
        text: "",
        prompt: false,
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
      document.getElementById("app").addEventListener("click", (e) => {
        alert("touchend");
      });
    },
    methods: {
      setupLeafletMap: function() {
        // this.mapDiv = L.map("mapContainer").fitWorld();
        this.mapDiv = L.map("mapContainer").setView(this.center, 13);
        L.tileLayer(`https://api.mapbox.com/styles/v1/mapbox/streets-v11/tiles/{z}/{x}/{y}?access_token=${this.accessToken}`).addTo(this.mapDiv);
        // this.mapDiv.locate({ setView: true, maxZoom: 16 });

        // this.mapDiv.on("locationfound", this.onLocationFound);
        // this.mapDiv.on("locationerror", this.onLocationError);

        var editableLayers = new L.FeatureGroup();
        this.mapDiv.addLayer(editableLayers);
        var drawnItems = new L.FeatureGroup();
        this.mapDiv.addLayer(drawnItems);

        L.Draw.Text = L.Draw.Marker.extend({
          initialize: function(map, options) {
            console.log(map, options);
            this.type = "Text";

            L.Draw.Feature.prototype.initialize.call(this, map, options);
          },

          addHooks: () => {
            this.text_prompt();
          },
        });

        L.Draw.Freeline = L.Draw.Polyline.extend({
          options: {
            repeatMode: false,
          },
          initialize: function(map, options) {
            this.type = "freeline";

            L.Draw.Feature.prototype.initialize.call(this, map, options);
          },

          addHooks: () => {
            this.start_drawing();

            if (this._map) {
              this._tooltip.updateContent({ text: "Click map start drawing custom shape." });
            }
          },
          removeHooks: () => {
            this.start_drawing();
          },
        });
        L.Draw.Arrow = L.Draw.Polyline.extend({
          options: {
            repeatMode: false,
          },
          initialize: function(map, options) {
            this.type = "arrow";

            L.Draw.Feature.prototype.initialize.call(this, map, options);
          },
        });

        // Settings custom toolbars before initilising the toolbar
        L.DrawToolbar.include({
          getModeHandlers: function(map) {
            return [
              {
                enabled: this.options.polyline,
                handler: new L.Draw.Polyline(map, this.options.polyline),
                title: L.drawLocal.draw.toolbar.buttons.polyline,
              },
              {
                enabled: this.options.polygon,
                handler: new L.Draw.Polygon(map, this.options.polygon),
                title: L.drawLocal.draw.toolbar.buttons.polygon,
              },
              {
                enabled: this.options.rectangle,
                handler: new L.Draw.Rectangle(map, this.options.rectangle),
                title: L.drawLocal.draw.toolbar.buttons.rectangle,
              },
              {
                enabled: this.options.circle,
                handler: new L.Draw.Circle(map, this.options.circle),
                title: L.drawLocal.draw.toolbar.buttons.circle,
              },
              {
                enabled: this.options.marker,
                handler: new L.Draw.Marker(map, this.options.marker),
                title: L.drawLocal.draw.toolbar.buttons.marker,
              },
              {
                enabled: true,
                handler: new L.Draw.Freeline(map, { icon: new L.Icon.Default() }),
                title: "Freeline",
              },
              {
                enabled: true,
                handler: new L.Draw.Text(map, { icon: new L.Icon.Default() }),
                title: "Text",
              },
              {
                enabled: true,
                handler: new L.Draw.Arrow(map, this.options.polygon),
                title: "Arrow",
              },
            ];
          },
        });

        var drawControl = new L.Control.Draw({
          draw: {
            polyline: {
              shapeOptions: {
                color: "#f357a1",
                weight: 10,
              },
            },
            arrow: {
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
          },
          edit: {
            featureGroup: drawnItems,
          },
        });

        this.mapDiv.addControl(drawControl);

        drawControl.setDrawingOptions({
          rectangle: {
            shapeOptions: {
              color: "#0000FF",
            },
          },
        });

        this.mapDiv.on(L.Draw.Event.CREATED, (e) => {
          console.log("CREATED", e);
          this.e = null;

          var type = e.layerType,
            layer = e.layer;

          if (type === "marker") {
            layer.bindPopup("A popup!");
          }
          if (type === "Text") {
            layer.bindPopup("A popup!");
          }

          editableLayers.addLayer(layer);
        });

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
          this.mapDiv.dragging.disable();
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

      text_prompt() {
        this.mapDiv.on("click ", (e) => {
          console.log(e);
          this.e = e;
          this.prompt = true;
        });
      },
      add_text_to_map() {
        // alert(this.text);
      },

      start_drawing() {
        if (L.Browser.mobile) {
          this.mapDiv.on("touchstart", (e) => {
            this.e = null;
            this.e = e;
            this.freehand_draw();
          });
        } else {
          this.mapDiv.on("mousedown", (e) => {
            this.e = null;
            this.e = e;
            this.freehand_draw();
          });
        }
      },
      freehand_draw(e) {
        if (!this.drawing) {
          this.drawing = true;

          this.line = L.polyline([]).addTo(this.mapDiv);
          this.line.addLatLng(this.e.latlng);
          console.log("ADDED LINE");
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

  .input {
    padding: 1rem;
    position: absolute;
    z-index: 1000;
  }
</style>
