<template>
  <div class="toasts">
    <div v-if="toast" class="toast" :class="toast.class">
      <app-icon :icon="toast.icon" />
      <span>{{ toast.message }}</span>
    </div>
  </div>
</template>

<script>
import AppIcon from './AppIcon';
export default {
  name: 'AppToast',

  components: { AppIcon },

  data() {
    return {
      toast: null,
    };
  },

  methods: {
    error(message) {
      this._show('error', message);
    },

    success(message) {
      this._show('success', message);
    },

    _show(style, message) {
      setTimeout(() => {
        this.toast = null;
      }, 2000);

      const icons = {
        success: 'check-circle',
        error: 'alert-circle',
      };

      this.toast = {
        message,
        style,
        icon: icons[style],
        class: `toast_${style}`,
      };
    },
  },
};
</script>

<style scoped>
.toasts {
  position: absolute; /* На самом деле тут должен быть Fixed */
  bottom: 8px;
  right: 8px;
  display: flex;
  flex-direction: column;
  justify-content: flex-end;
  white-space: pre-wrap;
  z-index: 999;
}

.toast {
  display: flex;
  flex: 0 0 auto;
  flex-direction: row;
  align-items: center;
  padding: 16px;
  background: #ffffff;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.15);
  border-radius: 4px;
  font-size: 18px;
  line-height: 28px;
  width: auto;
}

.toast + .toast {
  margin-top: 20px;
}

.toast > img {
  margin-right: 12px;
}

.toast.toast_success {
  color: var(--green);
}

.toast.toast_error {
  color: var(--red);
}

@media all and (min-width: 992px) {
  .toasts {
    bottom: 72px;
    right: 112px;
  }
}
</style>
