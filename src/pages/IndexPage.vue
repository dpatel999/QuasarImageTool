<template>
  <div id="app">
    <div id="label-bar">
      <h4>Your boxes</h4>
      <ul>
        <li
          v-for="(box, i) in boxes"
          :key="i"
          v-bind:class="{ active: i === activeBoxIndex }"
        >
          <input v-model="box.label" v-on:click="makeBoxActive(i)" />
          <a @click="removeBox(i)">x</a>
        </li>
      </ul>
    </div>
    <div
      id="image-wrapper"
      :style="{ backgroundImage: `url(test.jpg)` }"
      @mousedown="startDrawingBox"
      @mousemove="changeBox"
      @mouseup="stopDrawingBox"
      @touchstart="startDrawingBox"
      @touchmove="changeBox"
      @touchend="stopDrawingBox"
    >
      <BoundingBox
        v-if="drawingBox.active"
        :b-width="drawingBox.width"
        :b-height="drawingBox.height"
        :b-top="drawingBox.top"
        :b-left="drawingBox.left"
      />
      <BoundingBox
        v-for="(box, i) in boxes"
        :key="i"
        :b-top="box.top"
        :b-left="box.left"
        :b-label="box.label"
        :b-width="box.width"
        :b-height="box.height"
        :b-active="i === activeBoxIndex"
        :on-select="makeBoxActive"
        :b-index="i"
        :on-delete="removeBox"
        :on-resize-start="resizeStart"
        :on-resize-stop="resizeStop"
        @mousedown.stop="startDragging(i, $event)"
      />
    </div>
  </div>
</template>

<script>
import BoundingBox from "../components/BoundingBox.vue";
import { pick } from "lodash";

const getCoursorLeft = (e) => {
  return e.type === "touchstart" || e.type === "touchmove"
    ? e.pageX - e.target.offsetLeft
    : e.pageX;
};

const getCoursorTop = (e) => {
  return e.type === "touchstart" || e.type === "touchmove"
    ? e.pageY - e.target.offsetTop
    : e.pageY - 50;
};

export default {
  name: "app",
  components: { BoundingBox },
  data: function () {
    return {
      drawingBox: {
        active: false,
        top: 0,
        left: 0,
        height: 0,
        width: 0,
      },
      boxes: [],
      resizingBox: null,
      resizingHandle: null,
      draggingBox: null,
      dragStartX: 0,
      dragStartY: 0,
      dragging: false,
      activeBoxIndex: null,
      clickedBoxIndex: null,
    };
  },
  methods: {
    normalizeEvent(e) {
      if (e.touches) {
        e.pageX = e.touches[0].pageX;
        e.pageY = e.touches[0].pageY;
        e.clientX = e.touches[0].clientX;
        e.clientY = e.touches[0].clientY;
      }
      return e;
    },
    startDrawingBox(e) {
      e.preventDefault();
      e = this.normalizeEvent(e);
      this.drawingBox = {
        width: 0,
        height: 0,
        top: e.pageY - e.target.offsetTop,
        left: e.pageX - e.target.offsetLeft,
        active: true,
      };
    },
    changeBox(e) {
      if (this.drawingBox.active) {
        this.drawingBox = {
          ...this.drawingBox,
          width: getCoursorLeft(e) - this.drawingBox.left,
          height: getCoursorTop(e) - this.drawingBox.top,
        };
      }
    },
    stopDrawingBox() {
      if (this.drawingBox.active) {
        if (this.drawingBox.width > 5) {
          this.boxes.push({
            ...pick(this.drawingBox, ["width", "height", "top", "left"]),
          });
        }
        this.drawingBox = {
          active: false,
          top: 0,
          left: 0,
          height: 0,
          width: 0,
        };
      }
    },
    makeBoxActive(i) {
      if (!this.dragging) {
        this.activeBoxIndex = i;
      }
    },
    removeBox(i) {
      this.boxes = this.boxes.filter((elem, index) => {
        return index !== i;
      });
      this.activeBoxIndex = null;
    },
    onResizeStart(boxIndex, handleType) {
      this.resizingBox = {
        index: boxIndex,
        handleType: handleType,
        active: true,
      };
    },

    resizeStart(boxIndex, handle) {
      this.resizingBox = boxIndex;
      this.resizingHandle = handle;
    },

    resizeStop() {
      this.resizingBox = null;
      this.resizingHandle = null;
    },

    resizeBox(e) {
      if (this.resizingBox !== null) {
        const box = this.boxes[this.resizingBox];
        const handle = this.resizingHandle;
        const newLeft = getCoursorLeft(e);
        const newTop = getCoursorTop(e);

        // Resize depending on which handle was grabbed
        switch (handle) {
          case "top-left":
            box.width += box.left - newLeft;
            box.height += box.top - newTop;
            box.top = newTop;
            box.left = newLeft;
            break;
          case "top-right":
            box.width = newLeft - box.left;
            box.height += box.top - newTop;
            box.top = newTop;
            break;
          case "bottom-left":
            box.width += box.left - newLeft;
            box.height = newTop - box.top;
            box.left = newLeft;
            break;
          case "bottom-right":
            box.width = newLeft - box.left;
            box.height = newTop - box.top;
            break;
        }
      }
    },

    startDragging(index, event) {
      this.dragging = true;

      const delay = 200; // time in milliseconds
      this.dragTimeout = setTimeout(() => {
        this.draggingBox = index;
        this.dragStartX = event.clientX;
        this.dragStartY = event.clientY;
      }, delay);

      this.clickedBoxIndex = index;

      event.preventDefault();
    },

    handleDrag(event) {
      if (this.draggingBox !== null) {
        const box = this.boxes[this.draggingBox];
        box.left += event.clientX - this.dragStartX;
        box.top += event.clientY - this.dragStartY;
        this.dragStartX = event.clientX;
        this.dragStartY = event.clientY;
      }
    },

    stopDragging() {
      this.dragging = false;
      clearTimeout(this.dragTimeout);

      if (this.draggingBox === null) {
        this.makeBoxActive(this.clickedBoxIndex);
      }

      this.draggingBox = null;
      this.clickedBoxIndex = null;
    },
  },
  mounted() {
    window.addEventListener("mousemove", this.resizeBox);
    window.addEventListener("mouseup", this.resizeStop);
    window.addEventListener("mousemove", this.handleDrag);
    window.addEventListener("mouseup", this.stopDragging);

    window.addEventListener("touchstart", this.startDrawingBox);
    window.addEventListener("touchmove", this.changeBox);
    window.addEventListener("touchend", this.stopDrawingBox);
    window.addEventListener("touchmove", this.resizeBox);
    window.addEventListener("touchend", this.resizeStop);
    window.addEventListener("touchmove", this.handleDrag);
    window.addEventListener("touchend", this.stopDragging);
  },
  beforeUnmount() {
    window.removeEventListener("mousemove", this.resizeBox);
    window.removeEventListener("mouseup", this.resizeStop);
    window.removeEventListener("mousemove", this.handleDrag);
    window.removeEventListener("mouseup", this.stopDragging);

    window.removeEventListener("touchstart", this.startDrawingBox);
    window.removeEventListener("touchmove", this.changeBox);
    window.removeEventListener("touchend", this.stopDrawingBox);
    window.removeEventListener("touchmove", this.resizeBox);
    window.removeEventListener("touchend", this.resizeStop);
    window.removeEventListener("touchmove", this.handleDrag);
    window.removeEventListener("touchend", this.stopDragging);
  },
};
</script>

<style lang="scss" scoped>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #457fb8;

  #image-wrapper {
    height: 640px;
    width: 640px;
    background-repeat: no-repeat;
    position: relative;
  }
  #label-bar {
    float: left;
    margin-left: 700px;
    width: 220px;

    ul {
      padding: 0;

      li {
        list-style-type: none;
        padding: 8px 16px;
        &.active {
          background-color: lightblue;
        }

        a {
          cursor: pointer;
          display: inline-block;
          margin-left: 4px;
          font-weight: bold;
          color: red;
        }
      }
    }
  }
}
</style>
