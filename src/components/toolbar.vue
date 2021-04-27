<template>
  <div>
    <ul class="c-toolbar">
      <li :class="{ 'active': selected_tool == 'polygon' }" @click="update_selection('polygon')">
        <i class="fas fa-shapes"></i>
      </li>
      <li :class="{ 'active': selected_tool == 'text' }" @click="update_selection('text')">
        <i class="fab fa-amilia"></i>
      </li>
      <li :class="{ 'active': selected_type && selected_type.length }" @click="update_selection('pen')">
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
    </ul>
    <div class="c-toolbar is-color-picker">
      <p>
        <input type="color" id="favcolor" name="favcolor" v-model="color" />
        <i class="fas fa-dot-circle" @click="open_color_picker" :style="{ 'color': color }"></i>
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
        color: "#F6C8CC",
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
      select_tool_props(type) {
        // if (this.selected_tool) {
        this.selected_type = type;
        if (type == "line") {
          this.$emit("draw", { type: "line", color: this.color });
        } else if (type == "arrow") {
          this.$emit("draw", { type: "arrow", color: this.color });
        } else {
          this.$emit("draw", { type: "freehand", width: type, color: this.color });
        }
        // } else {
        //   this.$emit("stop-drawing");
        // }
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
    padding: 5px 0px !important;
    position: absolute;
    background: #3c3e42;
    color: white;
    border-radius: 3px;
    left: 22px;
    top: 160px;
    li {
      padding: 5px;
      cursor: pointer;
      position: relative;
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
      top: 0;
      left: 100%;
      height: 34px;
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
      color: pink;
      top: 290px;
      width: 25px;
      padding: 0 !important;
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
    // visibility: hidden;
    display: none;
  }
  .active {
    background: teal;
  }
</style>
