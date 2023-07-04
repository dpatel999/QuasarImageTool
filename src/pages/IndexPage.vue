<template>
  <div id="app" class="row">
    <div
      ref="imageWrapper"
      id="image-wrapper"
      @mousedown="startDrawingBox"
      @mousemove="changeBox"
      @mouseup="stopDrawingBox"
      @touchstart="startDrawingBox"
      @touchmove="changeBox"
      @touchend="stopDrawingBox"
    >
      <q-img
        src="test.jpg"
        :ratio="1"
        @load="handleImageLoad"
        spinner-color="primary"
      />
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
        @touchstart.stop="startDragging(i, $event)"
      />
    </div>
    <div id="label-bar">
      <h4>Your boxes</h4>
      <q-list dense bordered separator>
        <q-item
          v-for="(box, i) in boxes"
          :key="i"
          v-bind:class="{ active: i === activeBoxIndex }"
        >
          <q-item-section>
            <q-input
              v-model="box.label"
              v-on:click="makeBoxActive(i)"
              v-on:touchstart.stop="makeBoxActive(i)"
            />
            <q-btn
              flat
              round
              dense
              icon="close"
              @click="removeBox(i)"
              @touchstart="removeBox(i)"
            />
          </q-item-section>
        </q-item>
      </q-list>
    </div>
  </div>
</template>

<script>
import BoundingBox from "../components/BoundingBox.vue";
import { pick } from "lodash";

const getCoursorLeft = (e, vm) => {
  const targetRect = vm.$refs.imageWrapper.getBoundingClientRect();
  return (e.touches ? e.touches[0].clientX : e.clientX) - targetRect.left;
};

const getCoursorTop = (e, vm) => {
  const targetRect = vm.$refs.imageWrapper.getBoundingClientRect();
  return (e.touches ? e.touches[0].clientY : e.clientY) - targetRect.top;
};

import { QImg, QList, QItem, QItemSection, QInput, QBtn } from "quasar";

export default {
  name: "app",
  components: { QImg, QList, QItem, QItemSection, QInput, QBtn, BoundingBox },
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
      splitterModel: 50,
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
          width: getCoursorLeft(e, this) - this.drawingBox.left,
          height: getCoursorTop(e, this) - this.drawingBox.top,
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
    makeBoxActive(i, event) {
      if (event) event.preventDefault();
      if (!this.dragging) {
        this.activeBoxIndex = i;
      }
    },
    removeBox(i, event) {
      if (event) event.preventDefault();
      this.boxes = this.boxes.filter((elem, index) => {
        return index !== i;
      });
      this.activeBoxIndex = null;
    },
    removeMyself(event) {
      if (event) event.preventDefault();
      if (this.activeBoxIndex !== null) {
        this.boxes = this.boxes.filter(
          (_, index) => index !== this.activeBoxIndex
        );
        this.activeBoxIndex = null;
      }
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
        e = this.normalizeEvent(e);
        const box = this.boxes[this.resizingBox];
        const handle = this.resizingHandle;
        const newLeft = getCoursorLeft(e, this) - this.drawingBox.left;
        const newTop = getCoursorTop(e, this) - this.drawingBox.top;

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

      let clientX, clientY;
      if (event.touches) {
        clientX = event.touches[0].clientX;
        clientY = event.touches[0].clientY;
      } else {
        clientX = event.clientX;
        clientY = event.clientY;
      }

      this.dragTimeout = setTimeout(() => {
        this.draggingBox = index;
        this.dragStartX = clientX;
        this.dragStartY = clientY;
      }, delay);

      this.clickedBoxIndex = index;

      event.preventDefault();
    },

    handleDrag(event) {
      event = this.normalizeEvent(event);

      if (this.draggingBox !== null) {
        const box = this.boxes[this.draggingBox];

        let clientX, clientY;
        if (event.touches) {
          clientX = event.touches[0].clientX;
          clientY = event.touches[0].clientY;
        } else {
          clientX = event.clientX;
          clientY = event.clientY;
        }

        box.left += clientX - this.dragStartX;
        box.top += clientY - this.dragStartY;
        this.dragStartX = clientX;
        this.dragStartY = clientY;
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

    window.addEventListener("touchstart", this.startDrawingBox, {
      passive: false,
    });
    window.addEventListener("touchmove", this.changeBox, { passive: false });
    window.addEventListener("touchend", this.stopDrawingBox);
    window.addEventListener("touchmove", this.resizeBox, { passive: false });
    window.addEventListener("touchend", this.resizeStop);
    window.addEventListener("touchmove", this.handleDrag, { passive: false });
    window.addEventListener("touchend", this.stopDragging);
  },
  beforeUnmount() {
    window.removeEventListener("mousemove", this.resizeBox);
    window.removeEventListener("mouseup", this.resizeStop);
    window.removeEventListener("mousemove", this.handleDrag);
    window.removeEventListener("mouseup", this.stopDragging);

    window.removeEventListener("touchstart", this.startDrawingBox, {
      passive: false,
    });
    window.removeEventListener("touchmove", this.changeBox, { passive: false });
    window.removeEventListener("touchend", this.stopDrawingBox);
    window.removeEventListener("touchmove", this.resizeBox, { passive: false });
    window.removeEventListener("touchend", this.resizeStop);
    window.removeEventListener("touchmove", this.handleDrag, {
      passive: false,
    });
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
