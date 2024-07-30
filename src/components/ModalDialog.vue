<template>
  <div class="modal_window">
    <div class="modal_window_box">
      <slot></slot>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue'

export default defineComponent({
  name: 'ModalDialog',
  props: {
    show: {
      type: Boolean,
      default: false
    }
  }
})
</script>

<style scoped>
.modal_window {
  position: fixed;
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100vw;
  height: 100vh;
  left: 0;
  top: 0;
  z-index: 10;
}

.modal_window::before {
  content: '';
  position: absolute;
  display: block;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.2);
}

.modal_window_box {
  width: 300px;
  padding: 20px;
  background-color: #fff;
  border-radius: 20px;
  z-index: 1;
}

@media screen and (max-width: 400px) {
  .modal_window {
    width: calc(100% - 40px);
    padding: 0 20px;
  }
}

</style>