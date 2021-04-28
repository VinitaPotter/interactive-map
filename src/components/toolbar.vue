<template>
  <div>
    {{ active_toolbar }}
    <ul class="c-toolbar">
      <li
        :class="{ 'active': selected_type == 'polygon' || selected_type == 'rectangle' || selected_type == 'circle' }"
        @click="update_selection('polygon')"
      >
        <i class="fas fa-shapes"></i>
        <ul class="c-toolbar__nested" v-if="selected_tool == 'polygon'">
          <li @click="select_polygon_type('rectangle')">
            <i class="far fa-square"></i>
          </li>
          <li @click="select_polygon_type('polygon')">
            <i class="fas fa-draw-polygon"></i>
          </li>
          <li @click="select_polygon_type('circle')">
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
          <li @click="select_tool_props('5')">
            5
          </li>
          <li @click="select_tool_props('10')">
            10
          </li>
          <li @click="select_tool_props('20')">
            20
          </li>
          <li @click="select_tool_props('line')">
            <i class="fas fa-pencil-alt"></i>
          </li>
          <li @click="select_tool_props('arrow')">
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
          <li @click="select_marker_type('marker')">
            <i class="fas fa-map-marker-alt"></i>
          </li>
          <li @click="select_marker_type('image')">
            <i class="fas fa-image"></i>
          </li>
          <li @click="select_marker_type('camera')">
            <i class="fas fa-camera"></i>
          </li>
        </ul>
      </li>
    </ul>
    <div class="c-toolbar is-color-picker">
      <p>
        <i class="fas fa-dot-circle" @click="open_color_picker" :style="{ 'color': color }"></i>
        <input type="color" id="favcolor" name="favcolor" v-model="color" />
      </p>
    </div>
  </div>
</template>

<script>
  export default {
    data() {
      return {
        selected_tool: null,
        selected_type: null,
        color: "#F6C",
        active_toolbar: null,
      };
    },
    watch: {
      color(val) {
        this.update_color(val);
      },
    },

    methods: {
      update_color(col) {
        this.$emit("update-color", col);
      },
      update_selection(tool) {
        this.selected_tool == tool ? (this.selected_tool = null) : (this.selected_tool = tool);
      },
      select_polygon_type(type) {
        this.selected_type = type;
        this.tool_selected({ type: type, color: this.color });
      },
      select_marker_type(type) {
        this.selected_type = type;
        this.$emit("add-marker", type);
        this.active_toolbar = null;
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
          this.tool_selected({ type: "freehand", width: type, color: this.color });
        }
        this.active_toolbar = null;
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
      color: "#da1";
      top: 420px;
      width: 30px;
      padding: 3px 0px !important;
      height: 25px;
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
  #favcolor {
    visibility: hidden;
    // display: none;
  }
  .active {
    background: teal;
  }
</style>
