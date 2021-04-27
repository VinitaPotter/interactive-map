<template>
  <div>
    <toolbar @draw="trigger_toolbar($event)" @update-color="color = $event"></toolbar>
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
      };
    },
    components: {
      Toolbar,
    },

    mounted() {
      this.setupLeafletMap();
    },
    methods: {
      trigger_toolbar(payload) {
        this.width = payload.width;
        this.color = payload.color;
        switch (payload.type) {
          case "arrow":
            document.querySelector(".leaflet-draw-draw-arrow").click();
            break;
          case "line":
            document.querySelector(".leaflet-draw-draw-polyline").click();
            break;
          case "polygon":
            document.querySelector(".leaflet-draw-draw-polygon").click();
            break;
          case "text":
            document.querySelector(".leaflet-draw-draw-text").click();
            break;
          default:
            document.querySelector(".leaflet-draw-draw-freeline").click();
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

        // L.drawLocal = {
        //   draw: {
        //     toolbar: {
        //       // #TODO: this should be reorganized where actions are nested in actions
        //       // ex: actions.undo  or actions.cancel
        //       actions: {
        //         title: "Cancel - your text-",
        //         text: "- your text-",
        //       },
        //       finish: {
        //         title: "- your text-",
        //         text: "- your text-",
        //       },
        //       undo: {
        //         title: "- your text-",
        //         text: "- your text-",
        //       },
        //       buttons: {
        //         polyline: "- your text-",
        //         polygon: "- your text-",
        //         rectangle: "- your text-",
        //         circle: "- your text-",
        //         marker: "- your text-",
        //         circlemarker: "- your text-",
        //       },
        //     },
        //     handlers: {
        //       circle: {
        //         tooltip: {
        //           start: "- your text-",
        //         },
        //         radius: "- your text-",
        //       },
        //       circlemarker: {
        //         tooltip: {
        //           start: "- your text-.",
        //         },
        //       },
        //       marker: {
        //         tooltip: {
        //           start: "- your text-.",
        //         },
        //       },
        //       polygon: {
        //         tooltip: {
        //           start: "- your text-.",
        //           cont: "- your text-.",
        //           end: "- your text-.",
        //         },
        //       },
        //       polyline: {
        //         error: "<strong>Error:</strong> shape edges cannot cross!",
        //         tooltip: {
        //           start: "Click to start drawing line.",
        //           cont: "Click to continue drawing line.",
        //           end: "Click last point to finish line.",
        //         },
        //       },
        //       rectangle: {
        //         tooltip: {
        //           start: "- your text-.",
        //         },
        //       },
        //       simpleshape: {
        //         tooltip: {
        //           end: "Release mouse to finish drawing.",
        //         },
        //       },
        //       arrow: {
        //         tooltip: {
        //           cont: "Release mouse to finish drawing.",
        //         },
        //       },
        //     },
        //   },
        //   edit: {
        //     toolbar: {
        //       actions: {
        //         save: {
        //           title: "Save changes",
        //           text: "Save",
        //         },
        //         cancel: {
        //           title: "Cancel editing, discards all changes",
        //           text: "Cancel",
        //         },
        //         clearAll: {
        //           title: "Clear all layers",
        //           text: "Clear All",
        //         },
        //       },
        //       buttons: {
        //         edit: "Edit layers",
        //         editDisabled: "No layers to edit",
        //         remove: "Delete layers",
        //         removeDisabled: "No layers to delete",
        //       },
        //     },
        //     handlers: {
        //       edit: {
        //         tooltip: {
        //           text: "Drag handles or markers to edit features.",
        //           subtext: "Click cancel to undo changes.",
        //         },
        //       },
        //       remove: {
        //         tooltip: {
        //           text: "Click on a feature to remove.",
        //         },
        //       },
        //     },
        //   },
        // };

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
            repeatMode: false,
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
                color: this.color,
                weight: 10,
              },
            },
            polygon: {
              allowIntersection: false, // Restricts shapes to simple polygons
              drawError: {
                color: this.color, // Color the shape will turn when intersects
                message: "<strong>Polygon draw does not allow intersections!<strong> (allowIntersection: false)", // Message that will show when intersect
              },
              shapeOptions: {
                color: this.color,
              },
            },

            rectangle: {
              shapeOptions: {
                clickable: true,
                color: this.color,
              },
            },
            circle: {
              shapeOptions: {
                clickable: true,
                color: this.color,
              },
            },
          },
          edit: {
            featureGroup: this.drawnItems,
          },
        });

        this.mapDiv.addControl(this.drawControl);

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

          this.drawnItems.addLayer(layer);
          console.log("srawnitme", this.drawnItems);
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

      marker_promt() {
        this.mapDiv.on("click ", (e) => {
          if (this.selected_tool == "custommarker") {
            // <img class='image' src='${require("../assets/star.png")}' alt=' /><img class='image' src='~@/assets/image.png' alt=' /><img class='image' src='~@/assets/camera.png' alt=' />
            var myPopup = L.DomUtil.create("div");

            //   if (L.Browser.mobile) {
            //     myPopup.innerHTML = `
            //   <input id="image" type="file" style="display: none;" accept=".jpg, .jpeg, .png" /><button for="image" id="img-btn">Image</button>
            //  `;
            // } else {
            myPopup.innerHTML = `<div class='images'>
            <button id="icon">Icon</button>
            <input id="image" type="file" style="display: none;" accept=".jpg, .jpeg, .png" /><button for="image" id="img-btn">Image</button>
            <button id="camera-btn">Camera</button>
            </div>`;
            // }

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

      text_prompt() {
        this.mapDiv.on("click ", (e) => {
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
        if (this.selected_tool == "freeline") {
          if (!this.drawing) {
            this.drawing = true;

            this.line = L.polyline([]).addTo(this.drawnItems);
            this.line.addLatLng(this.e.latlng);
            console.log(this.width);
            this.line.setStyle({
              "color": this.color,
              "opacity": 0.9,
              "weight": this.width,
            });
            console.log("ADDED LINE");
          } else {
            this.stop_freehand();
          }
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
          console.log(this.polyline);
          if (this.selected_tool == "arrow") {
            console.log(e);
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
  .leaflet-draw-section {
    display: none;
  }
</style>
