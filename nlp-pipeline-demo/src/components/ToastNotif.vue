<script setup>
defineProps({
  show: { type: Boolean, default: false },
  message: { type: String, default: '' },
})
</script>

<template>
  <Teleport to="body">
    <Transition name="toast">
      <div class="toast" v-if="show">
        <div class="toast-icon">
          <svg viewBox="0 0 20 20" fill="none" width="18" height="18">
            <circle cx="10" cy="10" r="8" fill="var(--orange-500)"/>
            <path d="M6.5 10l2.5 2.5 4.5-4.5" stroke="white" stroke-width="1.6"
              stroke-linecap="round" stroke-linejoin="round"/>
          </svg>
        </div>
        <span class="toast-msg">{{ message }}</span>
      </div>
    </Transition>
  </Teleport>
</template>

<style scoped>
.toast {
  position: fixed;
  top: 24px;
  right: 24px;
  z-index: 200;
  display: flex;
  align-items: center;
  gap: 12px;
  background: var(--blue-900);
  color: #fff;
  padding: 14px 20px;
  border-radius: var(--radius-md);
  box-shadow: var(--shadow-xl);
  max-width: 400px;
  border-left: 3px solid var(--orange-500);
  font-size: 14px;
  line-height: 1.4;
}
.toast-icon { flex-shrink: 0; }
.toast-msg { color: rgba(255,255,255,0.92); }

.toast-enter-active { transition: all 0.35s cubic-bezier(0.16, 1, 0.3, 1); }
.toast-leave-active { transition: all 0.25s ease; }
.toast-enter-from { opacity: 0; transform: translateX(20px) translateY(-8px); }
.toast-leave-to   { opacity: 0; transform: translateY(-8px); }
</style>
