<template>
  <div>
    <div id="mapContainer"></div>
  </div>
</template>

<script>
import "leaflet/dist/leaflet.css";
import "leaflet-draw/dist/leaflet.draw.css";
// import "leaflet-defaulticon-compatibility/dist/leaflet-defaulticon-compatibility.webpack.css"; // Re-uses images from ~leaflet package

import * as L from "leaflet";
import "leaflet-draw";
import "leaflet-polylinedecorator";
// import "leaflet-defaulticon-compatibility";

export default {
  name: "Map",
  data() {
    return {
      popup: null,
      selected_tool: "",
      prompt: false,
      e: null,
      center: [51.505, -0.09],
      mapDiv: null,
      line: "",
      drawing: false,
      accessToken:
        "pk.eyJ1IjoidGVzdGVyMjM4Nzg2MjciLCJhIjoiY2tub2k2OWFrMHlxcDJ2bzVnbm5na213cyJ9.gpHnP345S2_zJHsj_Zx46A",
    };
  },
  mounted() {
    this.setupLeafletMap();
  },
  methods: {
    setupLeafletMap: function () {
      // this.mapDiv = L.map("mapContainer").fitWorld();
      this.mapDiv = L.map("mapContainer").setView(this.center, 13);
      L.tileLayer(
        `https://api.mapbox.com/styles/v1/mapbox/streets-v11/tiles/{z}/{x}/{y}?access_token=${this.accessToken}`
      ).addTo(this.mapDiv);
      // this.mapDiv.locate({ setView: true, maxZoom: 16 });

      // this.mapDiv.on("locationfound", this.onLocationFound);
      // this.mapDiv.on("locationerror", this.onLocationError);

      var editableLayers = new L.FeatureGroup();
      this.mapDiv.addLayer(editableLayers);
      var drawnItems = new L.FeatureGroup();
      this.mapDiv.addLayer(drawnItems);

      L.Draw.Text = L.Draw.Marker.extend({
        initialize: function (map, options) {
          this.type = "text";
          L.Draw.Feature.prototype.initialize.call(this, map, options);
        },

        addHooks: () => {
          this.text_prompt();
        },
        removeHooks: () => {
          this.text_prompt();
        },
      });

      L.Draw.Freeline = L.Draw.Polyline.extend({
        options: {
          repeatMode: false,
        },
        initialize: function (map, options) {
          this.type = "freeline";

          L.Draw.Feature.prototype.initialize.call(this, map, options);
        },

        addHooks: () => {
          this.start_drawing();

          if (this._map) {
            this._tooltip.updateContent({
              text: "Click map start drawing custom shape.",
            });
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
        initialize: function (map, options) {
          this.type = "arrow";

          L.Draw.Feature.prototype.initialize.call(this, map, options);
        },
        addHooks: () => {
          this.draw_arrow();

          if (this._map) {
            this._tooltip.updateContent({
              text: "Click map start drawing custom shape.",
            });
          }
        },
        removeHooks: () => {
          this.draw_arrow();
        },
      });

      var electricpole = L.Icon.extend({
        options: {
          shadowUrl: null,
          iconAnchor: new L.Point(12, 12),
          iconSize: new L.Point(30, 30),
          iconUrl: "images/marker-icon.png",
        },
      });

      // Settings custom toolbars before initilising the toolbar
      L.DrawToolbar.include({
        getModeHandlers: function (map) {
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
              handler: new L.Draw.Freeline(map, { icon: new electricpole() }),
              title: "Draw freehand shapes",
            },
            {
              enabled: true,
              handler: new L.Draw.Text(map, { icon: new L.Icon.Default() }),
              title: "Add text",
            },
            {
              enabled: true,
              handler: new L.Draw.Arrow(map, this.options.polygon),
              title: "Draw an arrow",
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
              color: "#3df", // Color the shape will turn when intersects
              message:
                "<strong>Polygon draw does not allow intersections!<strong> (allowIntersection: false)", // Message that will show when intersect
            },
            shapeOptions: {
              color: "#bad",
            },
          },

          rectangle: {
            shapeOptions: {
              clickable: true,
              color: "#ff0",
            },
          },
        },
        edit: {
          featureGroup: drawnItems,
        },
      });

      this.mapDiv.addControl(drawControl);

      this.mapDiv.on(L.Draw.Event.DRAWSTART, (e) => {
        console.log(e);
        this.selected_tool = e.layerType;
        console.log(this.selected_tool);
      });
      this.mapDiv.on(L.Draw.Event.DRAWSTOP, (e) => {
        this.selected_tool = "";
      });

      this.mapDiv.on(L.Draw.Event.CREATED, (e) => {
        console.log("mapdoc", this.mapDiv);
        this.e = null;

        var type = e.layerType,
          layer = e.layer;

        if (type === "marker") {
          layer.bindPopup("A popup!");
        }
        if (type === "text") {
          this.mapDiv.bindPopup("<p>Hello World</p>").openPopup();
          // layer.bindPopup("A popup!");
        }

        editableLayers.addLayer(layer);
      });

      this.mapDiv.on("mouseup", (e) => {
        this.e = null;
        this.stop_freehand();
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
      return function () {
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
        if (this.selected_tool == "Text") {
          var myPopup = L.DomUtil.create("div");
          myPopup.innerHTML =
            "<div><input class='input' id='value' placeholder='Add text' type='text'/></div><button  id='input-btn' >Submit</button>";
          this.popup = L.popup({ className: "input" })
            .setLatLng(e.latlng)
            .setContent(myPopup)
            .openOn(this.mapDiv);

          let button = L.DomUtil.get("input-btn");
          L.DomEvent.addListener(button, "click", () => {
            let content = L.DomUtil.get("value").value;
            if (content && content.length) {
              var marker = new L.marker(e.latlng, { opacity: 0.01 }); //opacity may be set to zero
              marker.bindTooltip(content, {
                permanent: true,
                className: "my-label",
                offset: [0, 0],
              });
              marker.addTo(this.mapDiv);
            }
            this.mapDiv.closePopup();
          });
        }
      });
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
      if (this.selected_tool == "freeline") {
        this.drawing = false;
        this.mapDiv.dragging.enable();
        this.mapDiv.touchZoom.enable();
        this.mapDiv.doubleClickZoom.enable();
        this.mapDiv.scrollWheelZoom.enable();
        this.mapDiv.boxZoom.enable();
        this.mapDiv.keyboard.enable();
        console.log("Closed line", this.line);
      }
    },
    draw_arrow() {
      console.log("drawing arrow");
      let pointA;
      let pointB;
      this.mapDiv.on("mousedown", (e) => {
        if (this.selected_tool == "arrow") {
          this.mapDiv.touchZoom.disable();
          this.mapDiv.dragging.disable();
          this.mapDiv.doubleClickZoom.disable();
          this.mapDiv.scrollWheelZoom.disable();
          this.mapDiv.boxZoom.disable();
          this.mapDiv.keyboard.disable();
          pointA = e.latlng;
        }
      });
      this.mapDiv.on("mouseup", (e) => {
        pointB = e.latlng;
        if (this.selected_tool == "arrow") {
          console.log(e);
          var polyline = L.polyline([pointA, pointB]).addTo(this.mapDiv);
          var decorator = L.polylineDecorator(polyline, {
            patterns: [
              {
                offset: "100%",
                repeat: 0,
                symbol: L.Symbol.arrowHead({
                  pixelSize: 15,
                  polygon: false,
                  pathOptions: { stroke: true },
                }),
              },
            ],
          }).addTo(this.mapDiv);
          this.mapDiv.touchZoom.enable();
          this.mapDiv.dragging.enable();
          this.mapDiv.doubleClickZoom.enable();
          this.mapDiv.scrollWheelZoom.enable();
          this.mapDiv.boxZoom.enable();
          this.mapDiv.keyboard.enable();
          pointB = null;
          pointA = null;
        }
      });
    },
  },
};
</script>

<style lang="scss">
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
.leaflet-popup-content {
  .input {
    padding: 0.5rem;
  }

  #input-btn {
    background: white;
    border: 1px solid #ccc;
    padding: 0.3rem 0.9rem;
    border-radius: 5px;
    margin-top: 5px;
  }
}
.leaflet-draw-draw-freeline {
  // background-position: 100% !important;
  background: url("~@/assets/pencil.png") no-repeat !important;
  background-color: #fff !important;
  background-position: center !important;
}
.leaflet-draw-draw-arrow {
  // background-position: unset !important;
  background: url("~@/assets/arrow.png") no-repeat !important;
  background-color: #fff !important;
  background-position: center !important;
}
.leaflet-draw-draw-text {
  background-position: 55% !important;
  background: url("~@/assets/font.png") no-repeat !important;
  background-color: #fff !important;
  background-position: center !important;
}
</style>
