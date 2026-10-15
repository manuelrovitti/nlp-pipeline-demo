<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

const props = defineProps({
  modelName: { type: String, default: 'Modello in esecuzione' },
  phases: { type: Array, default: () => [] },
  duration: { type: Number, default: 6000 }, // total ms
})
const emit = defineEmits(['done'])

const currentPhase = ref(0)
const progress = ref(0)
const progressLabel = ref(0)
const done = ref(false)

let rafId = null
let startTime = null
const phaseStep = computed => props.phases.length
  ? Math.floor((progress.value / 100) * props.phases.length)
  : 0

function tick(ts) {
  if (!startTime) startTime = ts
  const elapsed = ts - startTime
  const rawPct = Math.min(elapsed / props.duration, 1)
  // Ease-in-out-ish curve that slows near 100%
  const eased = rawPct < 0.85
    ? rawPct / 0.85 * 92
    : 92 + ((rawPct - 0.85) / 0.15) * 8
  progress.value = eased
  progressLabel.value = Math.floor(eased)

  if (props.phases.length) {
    currentPhase.value = Math.min(
      Math.floor((rawPct) * props.phases.length),
      props.phases.length - 1,
    )
  }

  if (rawPct < 1) {
    rafId = requestAnimationFrame(tick)
  } else {
    progress.value = 100
    progressLabel.value = 100
    currentPhase.value = props.phases.length - 1
    setTimeout(() => {
      done.value = true
      setTimeout(() => emit('done'), 600)
    }, 300)
  }
}

onMounted(() => { rafId = requestAnimationFrame(tick) })
onUnmounted(() => { if (rafId) cancelAnimationFrame(rafId) })
</script>

<template>
  <div class="loader-wrap" :class="{ 'is-done': done }">

    <!-- Model badge -->
    <div class="model-badge">
      <div class="model-dot" :class="{ 'is-pulsing': !done }" />
      <span>{{ modelName }}</span>
    </div>

    <!-- Token animation -->
    <div class="token-strip">
      <div
        v-for="n in 12"
        :key="n"
        class="token"
        :style="{
          animationDelay: `${(n - 1) * 0.12}s`,
          opacity: done ? 1 : undefined,
          background: done ? 'var(--orange-500)' : undefined,
        }"
      />
    </div>

    <!-- Progress bar -->
    <div class="progress-track">
      <div class="progress-bar" :style="{ width: progress + '%' }" />
    </div>
    <div class="progress-row">
      <span class="phase-label">
        <Transition name="phase-fade" mode="out-in">
          <span :key="currentPhase">
            {{ done ? '✓ Completato' : phases[currentPhase] || 'Elaborazione…' }}
          </span>
        </Transition>
      </span>
      <span class="progress-pct">{{ progressLabel }}%</span>
    </div>

    <!-- Phase steps -->
    <div class="phases-list" v-if="phases.length">
      <div
        v-for="(ph, i) in phases"
        :key="i"
        class="phase-row"
        :class="{
          'is-active': !done && i === currentPhase,
          'is-done':   done || i < currentPhase,
          'is-future':  !done && i > currentPhase,
        }"
      >
        <div class="phase-check">
          <svg v-if="done || i < currentPhase" viewBox="0 0 12 12" fill="none" width="10" height="10">
            <path d="M2 6l2.5 2.5 5.5-5" stroke="currentColor" stroke-width="1.6"
              stroke-linecap="round" stroke-linejoin="round"/>
          </svg>
          <div v-else-if="i === currentPhase" class="phase-spinner" />
          <div v-else class="phase-empty-dot" />
        </div>
        <span>{{ ph }}</span>
      </div>
    </div>

  </div>
</template>

<style scoped>
.loader-wrap {
  background: var(--surface);
  border: 1.5px solid var(--border);
  border-radius: var(--radius-lg);
  padding: 32px;
  display: flex;
  flex-direction: column;
  gap: 24px;
  box-shadow: var(--shadow-md);
  transition: border-color 0.4s;
}
.loader-wrap.is-done {
  border-color: var(--orange-400);
}

/* Model badge */
.model-badge {
  display: flex;
  align-items: center;
  gap: 10px;
  font-size: 13px;
  font-weight: 600;
  color: var(--text-2);
  background: var(--bg);
  padding: 8px 14px;
  border-radius: 99px;
  width: fit-content;
  border: 1px solid var(--border);
}
.model-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: var(--orange-500);
}
.model-dot.is-pulsing {
  animation: pulse-dot 1.4s ease-in-out infinite;
}
@keyframes pulse-dot {
  0%, 100% { opacity: 1; transform: scale(1); }
  50% { opacity: 0.45; transform: scale(0.75); }
}

/* Token strip — simulates token processing */
.token-strip {
  display: flex;
  gap: 5px;
  align-items: center;
}
.token {
  width: 36px;
  height: 10px;
  border-radius: 3px;
  background: var(--blue-100);
  animation: token-pulse 1.8s ease-in-out infinite;
}
@keyframes token-pulse {
  0%, 100% { background: var(--blue-100); }
  50%       { background: var(--orange-400); }
}
.token:nth-child(2n)   { width: 52px; }
.token:nth-child(3n)   { width: 28px; }
.token:nth-child(4n)   { width: 44px; }
.token:nth-child(6n)   { width: 60px; }

/* Progress */
.progress-track {
  height: 6px;
  background: var(--blue-100);
  border-radius: 99px;
  overflow: hidden;
}
.progress-bar {
  height: 100%;
  background: linear-gradient(90deg, var(--blue-500), var(--orange-500));
  border-radius: 99px;
  transition: width 0.15s linear;
}

.progress-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
}
.phase-label {
  font-size: 13px;
  color: var(--text-2);
  font-weight: 500;
}
.progress-pct {
  font-family: var(--font-head);
  font-size: 14px;
  font-weight: 700;
  color: var(--orange-600);
}

/* Phases list */
.phases-list {
  display: flex;
  flex-direction: column;
  gap: 10px;
  border-top: 1px solid var(--border);
  padding-top: 20px;
}
.phase-row {
  display: flex;
  align-items: center;
  gap: 10px;
  font-size: 13px;
  transition: color 0.2s;
}
.phase-row.is-active  { color: var(--text-1); font-weight: 500; }
.phase-row.is-done    { color: var(--text-3); }
.phase-row.is-future  { color: var(--text-3); }

.phase-check {
  width: 18px;
  height: 18px;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
}
.phase-row.is-done .phase-check  { color: var(--orange-500); }
.phase-row.is-active .phase-check { color: var(--blue-500); }

.phase-spinner {
  width: 12px;
  height: 12px;
  border: 2px solid var(--blue-100);
  border-top-color: var(--blue-500);
  border-radius: 50%;
  animation: spin 0.7s linear infinite;
}
@keyframes spin { to { transform: rotate(360deg); } }

.phase-empty-dot {
  width: 6px;
  height: 6px;
  border-radius: 50%;
  background: var(--border);
  margin: auto;
}

/* Phase text transition */
.phase-fade-enter-active, .phase-fade-leave-active { transition: all 0.2s; }
.phase-fade-enter-from { opacity: 0; transform: translateY(4px); }
.phase-fade-leave-to   { opacity: 0; transform: translateY(-4px); }
</style>
