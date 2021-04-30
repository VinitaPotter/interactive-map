<template>
  <div>
    <toolbar @draw="trigger_toolbar($event)" @update-color="color = $event" @add-marker="add_marker"></toolbar>
    <div class="photo-capture" id="camera">
      <div class="frame">
        <video id="player" class="is-hidden" controls autoplay></video>
        <canvas id="canvas" width="640" height="480" class=""></canvas>
        <div>
          <button id="capture" class="">Capture</button>
          <button id="save" class="is-hidden">Save</button>
          <button id="cancel" class="">Cancel</button>
        </div>
      </div>
    </div>
    <input id="image" type="file" style="display: none;" accept=".jpg, .jpeg, .png" />
    <button id="camera-btn" style="display: none;">Camera</button>
    <div id="mapContainer"></div>
  </div>
</template>

<script>
  import "leaflet/dist/leaflet.css";
  import "leaflet-draw/dist/leaflet.draw.css";
  import "leaflet-defaulticon-compatibility/dist/leaflet-defaulticon-compatibility.webpack.css"; // Re-uses images from ~leaflet package

  import * as L from "leaflet";
  import "leaflet-draw";
  import "leaflet-polylinedecorator";
  import "leaflet-defaulticon-compatibility";

  import Toolbar from "../components/toolbar.vue";

  export default {
    name: "Map",
    data() {
      return {
        arrowHandler: null,
        lineHandler: null,
        textHandler: null,
        circleHandler: null,
        rectangleHandler: null,
        polygonHandler: null,
        freehandHandler: null,

        drawControl: null,
        polyline: null,
        drawnItems: null,
        popup: null,
        selected_tool: "",
        prompt: false,
        e: null,
        center: [51.505, -0.09],
        mapDiv: null,
        line: "",
        drawing: false,
        accessToken: "pk.eyJ1IjoidGVzdGVyMjM4Nzg2MjciLCJhIjoiY2tuenBoNXRjMDd3NDJwb2JoZTgwdXkyNCJ9.zhWEouASkv8bAjhZ7xGxAw",
        width: 8,
        color: "#fea254",
        iconSize: [60, 60],
      };
    },
    components: {
      Toolbar,
    },

    mounted() {
      this.setupLeafletMap();
      this.arrowHandler = new L.Draw.Arrow(this.mapDiv);
      this.lineHandler = new L.Draw.Polyline(this.mapDiv, { repeatMode: true, shapeOptions: { color: this.color, opacity: 1 } });
      this.textHandler = new L.Draw.Text(this.mapDiv);
      this.circleHandler = new L.Draw.Circle(this.mapDiv, { repeatMode: true, shapeOptions: { color: this.color } });
      this.rectangleHandler = new L.Draw.Rectangle(this.mapDiv, { repeatMode: true, shapeOptions: { color: this.color } });
      this.polygonHandler = new L.Draw.Polygon(this.mapDiv, { repeatMode: true, shapeOptions: { color: this.color } });
      this.freehandHandler = new L.Draw.Freeline(this.mapDiv);
    },
    methods: {
      trigger_toolbar(payload) {
        this.width = payload.width;
        this.color = payload.color;

        switch (payload.type) {
          case "arrow":
            this.lineHandler.disable();
            this.textHandler.disable();
            this.circleHandler.disable();
            this.rectangleHandler.disable();
            this.polygonHandler.disable();
            this.freehandHandler.disable();
            this.arrowHandler.enable();
            break;
          case "line":
            this.arrowHandler.disable();
            this.textHandler.disable();
            this.circleHandler.disable();
            this.rectangleHandler.disable();
            this.polygonHandler.disable();
            this.freehandHandler.disable();

            this.lineHandler.options.shapeOptions.color = this.color;
            this.lineHandler.enable();
            break;
          case "text":
            this.arrowHandler.disable();
            this.lineHandler.disable();
            this.circleHandler.disable();
            this.rectangleHandler.disable();
            this.polygonHandler.disable();
            this.freehandHandler.disable();

            this.textHandler.enable();
            break;
          case "circle":
            this.arrowHandler.disable();
            this.lineHandler.disable();
            this.textHandler.disable();
            this.rectangleHandler.disable();
            this.polygonHandler.disable();
            this.freehandHandler.disable();

            this.circleHandler.options.shapeOptions.color = this.color;
            this.circleHandler.enable();
            break;
          case "rectangle":
            this.arrowHandler.disable();
            this.lineHandler.disable();
            this.textHandler.disable();
            this.circleHandler.disable();
            this.polygonHandler.disable();
            this.freehandHandler.disable();

            this.rectangleHandler.options.shapeOptions.color = this.color;
            this.rectangleHandler.enable();
            break;
          case "polygon":
            this.arrowHandler.disable();
            this.lineHandler.disable();
            this.textHandler.disable();
            this.circleHandler.disable();
            this.rectangleHandler.disable();
            this.freehandHandler.disable();

            this.polygonHandler.options.shapeOptions.color = this.color;
            this.polygonHandler.enable();
            break;
          case "freehand":
            this.arrowHandler.disable();
            this.lineHandler.disable();
            this.textHandler.disable();
            this.circleHandler.disable();
            this.rectangleHandler.disable();
            this.polygonHandler.disable();

            this.freehandHandler.enable();
            break;

          default:
            this.arrowHandler.disable();
            this.lineHandler.disable();
            this.textHandler.disable();
            this.circleHandler.disable();
            this.rectangleHandler.disable();
            this.polygonHandler.disable();
            this.freehandHandler.disable();
        }
      },
      setupLeafletMap: function() {
        // this.mapDiv = L.map("mapContainer").fitWorld();
        this.mapDiv = L.map("mapContainer").setView(this.center, 13);
        L.tileLayer(`https://api.mapbox.com/styles/v1/mapbox/streets-v11/tiles/{z}/{x}/{y}?access_token=${this.accessToken}`).addTo(this.mapDiv);
        // this.mapDiv.locate({ setView: true, maxZoom: 16 });

        // this.mapDiv.on("locationfound", this.onLocationFound);
        // this.mapDiv.on("locationerror", this.onLocationError);

        this.drawnItems = new L.FeatureGroup();
        this.mapDiv.addLayer(this.drawnItems);

        L.Draw.Text = L.Draw.Marker.extend({
          initialize: function(map, options) {
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
            repeatMode: true,
            showArea: false,
          },
          initialize: function(map, options) {
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
          deleteLastVertex: () => {
            let markers = this.line._latlngs;
            if (!markers.length) {
              return;
            } else {
              markers.pop();
              this.line.setLatLngs(markers);
            }
          },
          completeShape: function() {
            this.disable();
            if (this.options.repeatMode) {
              this.enable();
            }
          },
        });
        L.Draw.Arrow = L.Draw.Marker.extend({
          options: {
            repeatMode: false,
          },
          initialize: function(map, options) {
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
        L.Draw.CustomMarker = L.Draw.Marker.extend({
          initialize: function(map, options) {
            this.type = "custommarker";
            L.Draw.Feature.prototype.initialize.call(this, map, options);
          },

          addHooks: () => {
            this.marker_promt();
          },
          removeHooks: () => {
            this.marker_promt();
          },
          setOptions: function(options) {
            L.setOptions(this, options);
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
                handler: new L.Draw.CustomMarker(map, this.options.marker),
                title: L.drawLocal.draw.toolbar.buttons.marker,
              },
              {
                enabled: true,
                handler: new L.Draw.Freeline(map, this.options.polygon),
                title: "Draw freehand shapes",
              },
              {
                enabled: true,
                handler: new L.Draw.Text(map),
                title: "Add text",
              },
              {
                enabled: true,
                handler: new L.Draw.Arrow(map),
                title: "Draw an arrow",
              },
            ];
          },
        });

        this.drawControl = new L.Control.Draw({
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
                color: "#3df", // Color the shape will turn when intersects
                message: "<strong>Polygon draw does not allow intersections!<strong> (allowIntersection: false)", // Message that will show when intersect
              },
              shapeOptions: {
                color: "#bad",
              },
            },
            rectangle: {
              shapeOptions: {
                // clickable: true,
                color: "#ff0",
              },
            },
            circle: {
              shapeOptions: {
                // clickable: true,
                color: "#ff0",
              },
            },
          },
          edit: {
            featureGroup: this.drawnItems,
          },
        });

        this.mapDiv.addControl(this.drawControl);

        this.mapDiv.on(L.Draw.Event.DRAWSTART, (e) => {
          this.selected_tool = e.layerType;
        });
        this.mapDiv.on(L.Draw.Event.DRAWSTOP, (e) => {
          this.selected_tool = "";
        });

        this.mapDiv.on(L.Draw.Event.CREATED, (e) => {
          this.e = null;
          var type = e.layerType,
            layer = e.layer;

          if (type === "marker") {
            layer.bindPopup("A popup!");
          }
          this.drawnItems.addLayer(layer);
        });

        this.mapDiv.on("mouseup", (e) => {
          this.e = null;
          this.stop_freehand();
        });

        this.mapDiv.on("mousemove", this.throttle(this.drawLine, 25));
        this.mapDiv.on("zoomend", (e) => {
          let currentZoom = this.mapDiv.getZoom();
          let baseSize = 25;
          let updatedSize = baseSize + currentZoom * 2;

          Array.from(document.querySelectorAll(".my-div-icon")).forEach((el) => {
            let height = el.firstElementChild.style.height;

            el.firstElementChild.style.height = updatedSize + "px";
            el.firstElementChild.style.width = updatedSize + "px";
          });
        });
        this.mapDiv.on("dblclick", (e) => {
          if (this.rectangleHandler.enabled() || this.circleHandler.enabled()) {
            this.circleHandler.disable();
            this.rectangleHandler.disable();
          }
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

      marker_promt() {
        this.mapDiv.on("click ", (e) => {
          if (this.selected_tool == "custommarker") {
            var myPopup = L.DomUtil.create("div");

            myPopup.innerHTML = `<div class='images'>
                  <button id="icon">Icon</button>
                  <input id="image" type="file" style="display: none;" accept=".jpg, .jpeg, .png" /><button for="image" id="img-btn">Image</button>
                  <button id="camera-btn">Camera</button>
                  </div>`;

            this.popup = L.popup()
              .setLatLng(e.latlng)
              .setContent(myPopup)
              .openOn(this.mapDiv);

            //ICON MARKER
            let button = L.DomUtil.get("icon");
            let starIcon = L.icon({
              iconUrl: require("../assets/star.png"),
              // shadowUrl: require("../assets/star.png"),

              iconSize: [60, 60], // size of the icon
              // shadowSize: [50, 64], // size of the shadow
              iconAnchor: [50, 50], // point of the icon which will correspond to marker's location
              // shadowAnchor: [4, 62], // the same for the shadow
              popupAnchor: [-3, -76], // point from which the popup should open relative to the iconAnchor
            });

            // if (button) {
            L.DomEvent.addListener(button, "click", () => {
              var marker = new L.marker(e.latlng, { icon: starIcon }); //opacity may be set to zero

              marker.addTo(this.mapDiv);

              this.mapDiv.closePopup();
            });
            // }

            //FILE IMAGE MARKER
            let image_btn = L.DomUtil.get("img-btn");
            L.DomEvent.addListener(image_btn, "click", () => {
              let input = L.DomUtil.get("image");
              input.click();

              input.addEventListener("change", handleFiles, false);
              let that = this;
              function handleFiles() {
                const fileList = this.files;

                const reader = new FileReader();
                reader.addEventListener("load", (event) => {
                  let img = event.target.result;
                  let imageIcon = L.icon({
                    iconUrl: img,
                    iconSize: [60, 60], // size of the icon
                    iconAnchor: [50, 50], // point of the icon which will correspond to marker's location
                    popupAnchor: [-3, -76], // point from which the popup should open relative to the iconAnchor
                  });
                  var marker = new L.marker(e.latlng, { icon: imageIcon }); //opacity may be set to zero

                  marker.addTo(that.mapDiv);

                  that.mapDiv.closePopup();
                });
                reader.readAsDataURL(fileList[0]);
              }
            });

            //CAMERA ICON
            let camera_btn = L.DomUtil.get("camera-btn");
            // if (camera_btn) {
            L.DomEvent.addListener(camera_btn, "click", () => {
              const supported = "mediaDevices" in navigator;
              if (!supported) {
                alert("No camera found on this device");
                return;
              }
              const camera = document.getElementById("camera");

              const player = document.getElementById("player");
              const canvas = document.getElementById("canvas");
              const context = canvas.getContext("2d");
              const captureButton = document.getElementById("capture");
              const saveButton = document.getElementById("save");
              const cancelButton = document.getElementById("cancel");
              let photo;

              camera.classList.toggle("camera-active");

              player.classList.toggle("is-hidden");
              canvas.classList.toggle("is-hidden");

              const constraints = {
                video: true,
                audio: false,
              };

              // Get user media
              navigator.mediaDevices
                .getUserMedia(constraints)
                .then((stream) => {
                  player.srcObject = stream;
                })
                .catch(function(err) {
                  console.log("An error occurred: " + err);
                });

              captureButton.addEventListener("click", () => {
                // Draw the video frame to the canvas.
                context.drawImage(player, 0, 0, canvas.width, canvas.height);
                photo = canvas.toDataURL("image/png");
                player.classList.toggle("is-hidden");
                canvas.classList.toggle("is-hidden");
                captureButton.classList.toggle("is-hidden");
                saveButton.classList.toggle("is-hidden");
              });
              saveButton.addEventListener("click", () => {
                let photIcon = L.icon({
                  iconUrl: photo,
                  iconSize: [60, 60], // size of the icon
                  iconAnchor: [50, 50], // point of the icon which will correspond to marker's location
                  popupAnchor: [-3, -76], // point from which the popup should open relative to the iconAnchor
                });
                var marker = new L.marker(e.latlng, { icon: photIcon }); //opacity may be set to zero
                camera.classList.toggle("camera-active");

                marker.addTo(this.mapDiv);
                this.mapDiv.closePopup();

                // Stop all video streams.
                player.srcObject.getVideoTracks().forEach((track) => track.stop());
              });
              cancelButton.addEventListener("click", () => {
                camera.classList.toggle("camera-active");
                // Stop all video streams.
                player.srcObject.getVideoTracks().forEach((track) => track.stop());
              });
            });
            // }
          }
        });
      },

      add_marker(marker_type) {
        let action = (e) => {
          //ICON MARKER
          // https://stackoverflow.com/questions/46015066/leaflet-custom-icon-resize-on-zoom-performance-icon-vs-divicon
          if (marker_type == "marker") {
            let iconString = `<img class="img" src=${require("../assets/star.png")} alt/>`;
            let starIcon = L.divIcon({
              className: "my-div-icon",
              html: iconString,
            });
            var marker = new L.marker(e.latlng, { icon: starIcon }); //opacity may be set to zero
            marker.addTo(this.drawnItems);
            // this.mapDiv.off("click", this.offMap);
          }
          //FILE IMAGE MARKER
          if (marker_type == "image") {
            // let clicked_at = e;
            let input = document.getElementById("image");
            input.click();
            let onchange = (event1) => {
              const fileList = input.files;
              const reader = new FileReader();
              let readfile = (event2) => {
                let img = event2.target.result;
                console.log({ img });
                let iconString = `<img class="img" src=${img} alt/>`;
                let imgIcon = L.divIcon({
                  className: "my-div-icon",
                  html: iconString,
                });
                var marker = new L.marker(e.latlng, { icon: imgIcon }); //opacity may be set to zero
                marker.addTo(this.drawnItems);
                reader.removeEventListener("load", readfile);
              };
              reader.addEventListener("load", readfile);
              reader.readAsDataURL(fileList[0]);
              input.removeEventListener("change", () => {
                console.log("removed");
              });
              input.removeEventListener("change", onchange);
            };
            input.addEventListener("change", onchange);
          }
          //CAMERA ICON
          if (marker_type == "camera") {
            const supported = "mediaDevices" in navigator;
            if (!supported) {
              alert("No camera found on this device");
              return;
            }
            const camera = document.getElementById("camera");
            const player = document.getElementById("player");
            const canvas = document.getElementById("canvas");
            const context = canvas.getContext("2d");
            const captureButton = document.getElementById("capture");
            const saveButton = document.getElementById("save");
            const cancelButton = document.getElementById("cancel");
            let photo;
            camera.classList.toggle("camera-active");
            player.classList.toggle("is-hidden");
            canvas.classList.toggle("is-hidden");
            const constraints = {
              video: true,
              audio: false,
            };
            // Get user media
            navigator.mediaDevices
              .getUserMedia(constraints)
              .then((stream) => {
                player.srcObject = stream;
              })
              .catch(function(err) {
                console.log("An error occurred: " + err);
              });
            captureButton.addEventListener("click", () => {
              // Draw the video frame to the canvas.
              context.drawImage(player, 0, 0, canvas.width, canvas.height);
              photo = canvas.toDataURL("image/png");
              player.classList.toggle("is-hidden");
              canvas.classList.toggle("is-hidden");
              captureButton.classList.toggle("is-hidden");
              saveButton.classList.toggle("is-hidden");
            });
            saveButton.addEventListener("click", () => {
              let photIcon = L.icon({
                iconUrl: photo,
                iconSize: [60, 60], // size of the icon
                iconAnchor: [50, 50], // point of the icon which will correspond to marker's location
                popupAnchor: [-3, -76], // point from which the popup should open relative to the iconAnchor
              });
              var marker = new L.marker(e.latlng, { icon: photIcon }); //opacity may be set to zero
              camera.classList.toggle("camera-active");
              marker.addTo(this.drawnItems);
              this.mapDiv.closePopup();
              // Stop all video streams.
              player.srcObject.getVideoTracks().forEach((track) => track.stop());
            });
            cancelButton.addEventListener("click", () => {
              camera.classList.toggle("camera-active");
              // Stop all video streams.
              player.srcObject.getVideoTracks().forEach((track) => track.stop());
            });
          }
          // clicked_at = null;
          this.mapDiv.off("click ", action);
        };
        this.mapDiv.on("click", action);
        // this.mapDiv.off("click", action);
      },
      text_prompt() {
        let action = (e) => {
          if (this.selected_tool == "text") {
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
                  offset: [-50, 0],
                });
                marker.addTo(this.mapDiv);
              }
              this.mapDiv.closePopup();
              this.mapDiv.off("click ", action);
            });
          }
        };
        this.mapDiv.on("click ", action);
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
        if (this.selected_tool == "freeline") {
          if (!this.drawing) {
            this.drawing = true;

            this.line = L.polyline([]).addTo(this.drawnItems);
            this.line.addLatLng(this.e.latlng);
            this.line.setStyle({
              "color": this.color,
              "opacity": 0.9,
              "weight": this.width,
            });
          }
        }
      },
      stop_freehand() {
        if (this.selected_tool == "freeline") {
          console.log("stop freehand");
          this.drawing = false;
          this.mapDiv.dragging.enable();
          this.mapDiv.touchZoom.enable();
          this.mapDiv.doubleClickZoom.enable();
          this.mapDiv.scrollWheelZoom.enable();
          this.mapDiv.boxZoom.enable();
          this.mapDiv.keyboard.enable();
        }
      },
      drawLine(e) {
        if (this.drawing) {
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
      draw_arrow() {
        console.log("drawing arrow");
        let pointA;

        this.mapDiv.on("mousedown", (e) => {
          if (this.selected_tool == "arrow") {
            this.mapDiv.touchZoom.disable();
            this.mapDiv.dragging.disable();
            this.mapDiv.doubleClickZoom.disable();
            this.mapDiv.scrollWheelZoom.disable();
            this.mapDiv.boxZoom.disable();
            this.mapDiv.keyboard.disable();
            pointA = e.latlng;
            this.polyline = L.polyline([pointA, pointA]).addTo(this.mapDiv);
            this.polyline.setStyle({
              "color": this.color,
              "opacity": 1,
              "weight": 2,
            });
          }
        });
        this.mapDiv.on("mousemove", (e) => {
          if (this.selected_tool == "arrow" && this.polyline) {
            this.polyline.setLatLngs([pointA, e.latlng]);
          }
        });
        this.mapDiv.on("mouseup", (e) => {
          if (this.selected_tool == "arrow") {
            var decorator = L.polylineDecorator(this.polyline, {
              patterns: [
                {
                  offset: "100%",
                  repeat: 0,
                  symbol: L.Symbol.arrowHead({
                    pixelSize: 15,
                    polygon: false,
                    pathOptions: { stroke: true, color: this.color },
                  }),
                },
              ],
            }).addTo(this.drawnItems);
            this.mapDiv.touchZoom.enable();
            this.mapDiv.dragging.enable();
            this.mapDiv.doubleClickZoom.enable();
            this.mapDiv.scrollWheelZoom.enable();
            this.mapDiv.boxZoom.enable();
            this.mapDiv.keyboard.enable();

            pointA = null;
            this.polyline = null;
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
  .leaflet-draw-draw-custommarker {
    // background-position: 100% !important;
    background: url("~@/assets/cam.png") no-repeat !important;
    background-color: #fff !important;
    background-position: center !important;
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

  .image {
    height: 1.5rem;
    width: 1.5rem;
  }

  .images {
    background: white;
  }

  button {
    background: white;
    border: 1px solid #ddd;
    padding: 0.3rem 0.7rem;
    margin: 0.5rem;
    border-radius: 2px;
    cursor: pointer;
  }
  .photo-capture {
    position: absolute;
    z-index: 10001;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    width: 100%;
    visibility: hidden;

    &.camera-active {
      visibility: visible;
      top: 0;
    }
    .frame {
      background: white;
      padding: 1rem;
      box-shadow: 10px 10px 10px #aaa;
      border-radius: 11px;
      padding-top: 0;
    }
  }
  .is-vhidden {
    visibility: hidden !important;
  }
  .is-hidden {
    display: none;
  }
  .leaflet-draw-toolbar-top {
    display: none;
  }

  .leaflet-draw-edit-remove {
    background: #3c3e42 !important;
    &::before {
      content: "\f2ed";
      color: white;
      font-weight: 900;
      font-family: FontAwesome;
    }
  }
  .leaflet-draw-edit-edit {
    background: #3c3e42 !important;
    &::before {
      content: "\f044";
      color: white;
      font-weight: 900;
      font-family: FontAwesome;
    }
  }
  .leaflet-draw-actions {
    // display: none !important;
  }

  .my-label {
    background: none;
    box-shadow: none;
    font-weight: 600;
    font-size: 1.5rem;
    color: maroon;
  }

  .my-div-icon {
    height: 60px;
    width: 60px;
    // background: blue;
  }
  .img {
    height: 60px;
    width: 60px;
  }
</style>
