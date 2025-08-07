<template>
  <span
    class="dialog"
    v-if="modelValue"
    :style="{
      cursor: persistent ? 'default' : 'pointer',
    }"
    @click="chiudiDialog"
  >
    <span
      class="dialog-content"
      @click.stop
      :style="{
        cursor: 'default',
        display: 'inline-block',
        width: width,
      }"
    >
      <slot></slot>
    </span>
  </span>
</template>
<script>
export default {
  emits: ["change", "update:modelValue"],
  props: {
    modelValue: {
      type: Boolean,
      default: false,
    },
    persistent: {
      type: Boolean,
      default: false,
    },
    width: {
      type: String,
      default: "auto",
    },
  },
  watch: {
    modelValue(newValue) {
      if (newValue) {
        document.body.classList.add("body-no-scroll");
      } else {
        document.body.classList.remove("body-no-scroll");
      }
    },
  },
  methods: {
    chiudiDialog() {
      if (!this.persistent) {
        this.$emit("update:modelValue", false);
      }
    },
  },
};
</script>
<style scoped>
.dialog {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.5);
  z-index: 10000;
  touch-action: none;
  display: flex;
  justify-content: center;
  align-items: center;
}
.dialog-content {
  display: inline-block;
  background: transparent;
  max-width: 90%;
  min-width: 200px;
  transform: none;
}
.body-no-scroll {
  overflow: hidden;
  height: 100%;
  width: 100%;
}
</style>