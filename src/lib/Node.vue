<template>
  <g>
    <g v-if="editable">
      <text
        v-if="selected"
        :x="x + 5"
        :y="y + node.height + 22"
        class="button"
        fill="#00b894"
        @click="editCandidate"
      >
        {{ labels.edit || "Edit" }}
      </text>
      <text
        v-if="selected"
        :x="x + 5"
        :y="y - 10"
        class="button"
        fill="#00b894"
        stroke="none"
        @click="toggleSelect"
      >
        {{
          !createLinkMode ? labels.link || "Link" : labels.cancel || "Cancel"
        }}
      </text>
      <text
        v-if="selected"
        :x="x + 100"
        :y="y - 10"
        class="button"
        fill="orange"
        @click="copy"
      >
        {{ labels.copy || "Copy" }}
      </text>
      <text
        v-if="selected"
        :x="x + 65"
        :y="y + node.height + 22"
        class="button"
        fill="#ff7675"
        @click="remove"
      >
        {{ labels.remove || "Remove" }}
      </text>
      <text
        v-if="createLinkMode && !selected"
        :x="x + 5"
        :y="y - 10"
        class="button"
        fill="#ff7675"
        @click="commitDest"
      >
        {{ labels.select || "Select" }}
      </text>
    </g>
    <ellipse
      v-if="node.shape === 'ellipse'"
      class="grab"
      :cx="x + node.width / 2"
      :cy="y + node.height / 2"
      :width="node.width"
      :height="node.height"
      :rx="node.width / 2"
      :ry="node.height / 2"
      :fill="node.color && node.color.hexa ? node.color.hexa : '#ecf0f1'"
      :stroke-width="node.strokeWeight"
      :stroke="node.stroke"
      @touchstart="mousedown"
      @mousedown="mousedown"
      @mousemove="mousemove"
      @touchmove="mousemove"
      @mouseup="mouseup"
      @touchend="mouseup"
    />
    <rect
      v-else-if="node.shape === 'rectangle'"
      class="grab"
      :x="x"
      :y="y"
      :width="node.width"
      :height="node.height"
      rx="5"
      ry="3"
      :fill="node.color && node.color.hexa ? node.color.hexa : '#ecf0f1'"
      :stroke-width="node.strokeWeight"
      :stroke="node.stroke"
      @touchstart="mousedown"
      @mousedown="mousedown"
      @mousemove="mousemove"
      @touchmove="mousemove"
      @mouseup="mouseup"
      @touchend="mouseup"
    />
	  <rect
      v-else-if="node.shape === 'text' || node.shape === 'title'"
      class="grab"
      :x="x"
      :y="y"
      :width="node.width"
      :height="node.height"
			:font-size="node.font_size"
      rx="5"
      ry="3"
      fill="#FF000000"
      :stroke-width="node.strokeWeight"
      :stroke="node.stroke"
      @touchstart="mousedown"
      @mousedown="mousedown"
      @mousemove="mousemove"
      @touchmove="mousemove"
      @mouseup="mouseup"
      @touchend="mouseup"
    />
	  <a target="_blank" :href="url">
      <text
        :x="x + node.width / 2"
        :y="y + node.height / 2"
        :fill="node.font_color && node.font_color.hexa ? node.font_color.hexa : '#34495e'"
        font-family="Meiryo UI, sans-serif"
        :font-size="node.font_size ? node.font_size : 20"
        text-anchor="middle"
      >
        {{ node.text }}
      </text>
    </a>
  </g>
</template>
<script>
import mouseLocation from "../mouseLocation";
export default {
  mixins: [mouseLocation],
  props: {
    node: {
      width: Number,
      height: Number,
      id: String,
      point: {
        type: Object,
        default: {
          x: 0,
          y: 0
        }
      },
      content: {
        text: String,
        url: String,
        color: String
      },
      shape: {
        type: String,
        default: "rectangle"
      },
      stroke: String,
      strokeWeight: Number,
		font_size: Number
    },
    editable: Boolean,
    createLinkMode: Boolean,
    selected: Boolean,
    labels: Object,
    scale: String,
    rWidth: Number,
    rHeight: Number
  },
  watch: {
    node() {
      this.x = this.node.point.x;
      this.y = this.node.point.y;
    }
  },
  data() {
    return {
      startPosition: null,
      cursorOffset: {
        x: 0,
        y: 0
      },
      id: this.node.id,
      x: this.node.point.x,
      y: this.node.point.y,
      content: this.node.content
    };
  },
  methods: {
    toggleSelect() {
      this.$emit("toggleSelect");
    },
    commitDest() {
      this.$emit("commitDest", this.id);
    },
    remove() {
      this.$emit("remove", this.id);
    },
    copy() {
      this.$emit("copy", this.node);
    },
    mousedown(e) {
      this.$emit("click", this.id);
      if (!this.editable) return;
      this.$emit("select", this.id);
      const [x, y] = this.getLocation(e);
      this.cursorOffset.x = x;
      this.cursorOffset.y = y;
      this.startPosition = { x: this.x, y: this.y };
this.editCandidate()
      document.addEventListener("mousemove", this.mousemove);
      document.addEventListener("mouseup", this.mouseup);
    },
    mousemove(e) {
      if (this.startPosition) {
        e.preventDefault();
        const [x, y] = this.getLocation(e);
        this.x =
          this.startPosition.x +
          (x - this.cursorOffset.x) / this.rWidth / parseFloat(this.scale);
        this.y =
          this.startPosition.y +
          (y - this.cursorOffset.y) / this.rHeight / parseFloat(this.scale);
        this.$emit("updateLocation", {
          id: this.id,
          x: this.x,
          y: this.y
        });
      }
    },
    mouseup() {
      this.startPosition = null;

      document.removeEventListener("mousemove", this.mousemove);
      document.removeEventListener("mouseup", this.mouseup);
    },
    editCandidate() {
      this.$emit("editNode", {
        id: this.id,
		  text: this.node.text,
		  color: this.node.color,
		  font_color: this.node.font_color,
		  font_size: this.node.font_size,
        shape: this.node.shape,
        width: this.node.width,
        height: this.node.height,
        stroke: this.node.stroke,
        strokeWeight: this.node.strokeWeight
      });
    }
  }
};
</script>
<style lang="scss" scoped>
.shadow {
  filter: drop-shadow(1px 1px 3px rgba(0, 0, 0, 0.3));
  -webkit-filter: drop-shadow(1px 1px 3px rgba(0, 0, 0, 0.3));
  -moz-filter: drop-shadow(1px 1px 3px rgba(0, 0, 0, 0.3));
}
.button {
  cursor: pointer;
}
</style>
