<template>
  <div class="box-wrapper">
    <div
      class="label"
      v-if="bLabel"
      :style="{
        top: bTop - 10 + 'px',
        left: bLeft + 'px',
        width: bWidth + 4 + 'px',
      }"
    >
      {{ bLabel }}
    </div>

    <a
      class="box-delete"
      v-on:click="removeMyself"
      v-if="bActive"
      :style="{
        top: bTop - 18 + 'px',
        left: bLeft + bWidth + 'px',
      }"
    >
      x
    </a>
    <div
      class="box"
      :style="{
        top: bTop + 'px',
        left: bLeft + 'px',
        width: bWidth + 'px',
        height: bHeight + 'px',
      }"
      v-bind:class="{ active: bActive }"
      @mousedown="selectBox"
    >
      <div
        class="handle top-left"
        @mousedown="startResizing('top-left', $event)"
      ></div>
      <div
        class="handle top-right"
        @mousedown="startResizing('top-right', $event)"
      ></div>
      <div
        class="handle bottom-left"
        @mousedown="startResizing('bottom-left', $event)"
      ></div>
      <div
        class="handle bottom-right"
        @mousedown="startResizing('bottom-right', $event)"
      ></div>
    </div>
  </div>
</template>

<script>
export default {
  name: "BoundingBox",
  props: [
    "b-top",
    "b-left",
    "b-width",
    "b-height",
    "on-select",
    "b-active",
    "b-index",
    "on-delete",
    "b-label",
    "on-resize-start",
    "on-resize-stop",
  ],
  methods: {
    selectBox() {
      this.onSelect(this.bIndex);
    },
    removeMyself() {
      this.onDelete(this.bIndex);
    },
    startResizing(handleType, event) {
      // prevent event propagation to avoid selecting the box when starting to resize
      event.stopPropagation();
      this.onResizeStart(this.bIndex, handleType);
    },
  },
};
</script>

<style lang="scss" scoped>
.box {
  position: absolute;
  border: 2px #90ee90 solid;

  &:hover,
  &.active {
    background-color: rgba(144, 238, 144, 0.2);
  }

  z-index: 3;
}
.box-delete {
  position: absolute;
  z-index: 6;
  font-weight: bold;
  color: red;
  cursor: pointer;
  font-size: 24px;
  font-weight: bold;
}
.label {
  position: absolute;
  height: 22px;
  font-size: 16px;
  color: #000;
  font-weight: bold;
  background-color: #90ee90;
  z-index: 4;
}
.handle {
  position: absolute;
  width: 10px;
  height: 10px;
  background-color: red;
  cursor: pointer;
}
.top-left {
  top: -5px;
  left: -5px;
}
.top-right {
  top: -5px;
  right: -5px;
}
.bottom-left {
  bottom: -5px;
  left: -5px;
}
.bottom-right {
  bottom: -5px;
  right: -5px;
}
</style>
