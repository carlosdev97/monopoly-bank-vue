<template>
  <div
    ref="toast"
    :class="[
      'toast',
      'align-items-center',
      'border-0',
      'position-fixed',
      'bottom-0',
      'end-0',
      'm-3',
      color,
    ]"
    role="alert"
    aria-live="assertive"
    aria-atomic="true"
  >
    <div className="d-flex">
      <div className="toast-body">
        {{ message }}
      </div>
      <button
        type="button"
        className="btn-close btn-close-white me-2 m-auto"
        @click="hideToast"
        aria-label="Close"
      ></button>
    </div>
  </div>
</template>

<script>
import { Toast } from "bootstrap";

export default {
  props: {
    message: {
      type: String,
      default: "Hello, world! This is a toast message.",
    },
    color: {
      type: String,
      default: "bg-primary",
    },
  },
  mounted() {
    this.toastInstance = new Toast(this.$refs.toast);
  },
  methods: {
    showToast() {
      this.toastInstance.show();
    },
    hideToast() {
      this.toastInstance.hide();
    },
  },
};
</script>

<style scoped>
/* Puedes personalizar los estilos aquí si es necesario */
</style>
