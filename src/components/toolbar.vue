<template>
  <div>
    <ul class="c-toolbar">
      <li
        :class="{ 'active': selected_type == 'polygon' || selected_type == 'rectangle' || selected_type == 'circle' }"
        @click="update_selection('polygon')"
      >
        <i class="fas fa-shapes"></i>
        <ul class="c-toolbar__nested" v-if="selected_tool == 'polygon'">
          <li @click="select_polygon_type('rectangle')" :class="{ 'active': selected_type == 'rectangle' }">
            <i class="far fa-square"></i>
          </li>
          <li @click="select_polygon_type('polygon')" :class="{ 'active': selected_type == 'polygon' }">
            <i class="fas fa-draw-polygon"></i>
          </li>
          <li @click="select_polygon_type('circle')" :class="{ 'active': selected_type == 'circle' }">
            <i class="far fa-circle"></i>
          </li>
        </ul>
      </li>
      <li :class="{ 'active': selected_tool == 'text' }" @click="update_selection('text')">
        <i class="fab fa-amilia"></i>
      </li>
      <li :class="{ 'active': selected_type == 'line' || selected_type == 'arrow' || selected_type == 'freehand' }" @click="update_selection('pen')">
        <i class="fas fa-pen"></i>
        <ul class="c-toolbar__nested" v-if="selected_tool == 'pen'">
          <li @click="select_tool_props('5')" :class="{ 'active': width == '5' && selected_type == 'freehand' }">
            5
          </li>
          <li @click="select_tool_props('10')" :class="{ 'active': width == '10' && selected_type == 'freehand' }">
            10
          </li>
          <li @click="select_tool_props('20')" :class="{ 'active': width == '20' && selected_type == 'freehand' }">
            20
          </li>
          <li @click="select_tool_props('line')" :class="{ 'active': selected_type == 'line' }">
            <i class="fas fa-pencil-alt"></i>
          </li>
          <li @click="select_tool_props('arrow')" :class="{ 'active': selected_type == 'arrow' }">
            <i class="fas fa-long-arrow-alt-up"></i>
          </li>
        </ul>
      </li>
      <li
        :class="{ 'active': selected_type == 'marker' || selected_type == 'image' || selected_type == 'camera' }"
        @click="update_selection('marker')"
      >
        <i class="fas fa-map-pin"></i>
        <ul class="c-toolbar__nested" v-if="selected_tool == 'marker'" style="height:33px;top:0px">
          <li @click="select_marker_type('marker')" :class="{ 'active': selected_type == 'marker' }">
            <i class="fas fa-map-marker-alt"></i>
          </li>
          <li @click="select_marker_type('image')" :class="{ 'active': selected_type == 'image' }">
            <i class="fas fa-image"></i>
          </li>
          <!-- <li @click="select_marker_type('camera')">
            <i class="fas fa-camera"></i>
          </li> -->
        </ul>
      </li>
    </ul>
    <div class="c-toolbar is-color-picker">
      <!-- <p>
        <i class="fas fa-dot-circle" @click="open_color_picker" :style="{ 'color': color }"></i>
      </p> -->
      <input type="color" id="favcolor" name="favcolor" v-model="color" @change="update_color" />
    </div>
  </div>
</template>

<script>
  export default {
    data() {
      return {
        selected_tool: null,
        selected_type: null,
        color: "#000",
        width: -1,
      };
    },
    // watch: {
    //   color(val) {
    //     this.update_color(val);
    //   },
    // },

    methods: {
      update_color(col) {
        this.$emit("update-color", col);
        if (this.selected_type != "freeline") {
          this.$emit("draw", { type: this.selected_type, color: this.color });
        }
      },
      update_selection(tool) {
        this.selected_tool == tool ? (this.selected_tool = null) : (this.selected_tool = tool);
        if (this.selected_tool == "text") {
          this.selected_type = null;
          this.$emit("draw", { type: "text", color: this.color });
        }
      },
      select_polygon_type(type) {
        this.selected_type = type;
        this.tool_selected({ type: type, color: this.color });
      },
      select_marker_type(type) {
        this.selected_type = type;
        this.$emit("add-marker", type);
      },
      select_tool_props(type) {
        if (type == "line") {
          this.selected_type = "line";
          this.tool_selected({ type: "line", color: this.color });
        } else if (type == "arrow") {
          this.selected_type = "arrow";
          this.tool_selected({ type: "arrow", color: this.color });
        } else {
          this.selected_type = "freehand";
          this.width = type;
          this.tool_selected({ type: "freehand", width: type, color: this.color });
        }
      },
      tool_selected(payload) {
        this.$emit("draw", payload);
      },
      open_color_picker() {
        document.getElementById("favcolor").click();
      },
    },
  };
</script>

<style lang="scss">
  .c-toolbar {
    z-index: 1000000;
    list-style: none;
    padding: 0px !important;
    position: absolute;
    background: #3c3e42;
    color: white;
    border-radius: 3px;
    left: 20px;
    top: 250px;
    width: 31px;
    li {
      padding: 7px;
      cursor: pointer;
      position: relative;
      margin: 1px;
      &:not(:last-of-type) {
        border-bottom: 1px solid #777;
      }
    }
    &__nested {
      list-style: none;
      padding: 0 !important;
      display: inline-flex;
      position: absolute;
      background: #3c3e42;
      top: -1px;
      left: 100%;
      height: 35px;

      li {
        padding: 10px;
        cursor: pointer;
        align-items: center;
        justify-content: center;
        font-size: small;
        font-weight: 900;
        &:not(:last-of-type) {
          border-bottom: 1px solid #777;
        }
      }
    }
    &.is-color-picker {
      // color: "#da1";
      top: 420px;
      // width: 30px;
      // padding: 3px 0px !important;
      // height: 25px;
      p {
        margin: auto;
        padding: 5px;
        cursor: pointer;
      }
    }
  }
  li {
    color: white;
  }
  .fa-long-arrow-alt-up {
    transform: rotate(330deg);
  }
  // #favcolor {
  //   visibility: hidden;
  //   // display: none;
  // }
  // @media only screen and (max-width: 600px) {
  //   #favcolor {
  //     visibility: visible;
  //     width: 25px;
  //     // display: none;
  //   }
  //   .is-color-picker {
  //     background: transparent;
  //   }
  //   .fa-dot-circle {
  //     display: none;
  //   }
  // }
  #favcolor {
    width: 25px;
    padding: 2px;
    border-radius: 7px;
  }
  .active {
    background: teal;
  }

  input[type="color"] {
    background: #3c3e42;
  }

  input[type="color"]::-webkit-color-swatch {
    border: none;
    border-radius: 50%;
    padding: 0;
  }

  input[type="color"]::-webkit-color-swatch-wrapper {
    border: none;
    border-radius: 50%;
    padding: 4px;
  }
</style>
