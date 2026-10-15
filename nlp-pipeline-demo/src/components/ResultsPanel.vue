<script setup>
import { ref, computed } from 'vue'

const props = defineProps({
  files: { type: Array, default: () => [] },
})

const showModal = ref(false)

const mockResults = computed(() => {
  const fileNames = props.files.length
    ? props.files.map(f => f.name)
    : ['documento_1.pdf', 'documento_2.pdf', 'documento_3.pdf']

  return fileNames.map((name) => ({
    file: name,
    confidence: (0.80 + Math.random() * 0.20).toFixed(2),
    tokens: Math.floor(800 + Math.random() * 2400),
    entities: Math.floor(12 + Math.random() * 40),
    status: 'Classificato',
  }))
})

const avgConfidence = computed(() => {
  const sum = mockResults.value.reduce((a, r) => a + parseFloat(r.confidence), 0)
  return (sum / mockResults.value.length * 100).toFixed(0)
})

function downloadCSV() {
  const a = document.createElement('a')
  a.href = '/demo/risultati.csv'
  a.download = 'risultati.csv'
  a.click()
}

function confidenceColor(c) {
  const n = parseFloat(c)
  if (n >= 0.85) return '#15803d'
  if (n >= 0.70) return '#b45309'
  return '#b91c1c'
}
</script>

<template>
  <div class="results-wrap">

    <!-- Stats row -->
    <div class="stats-row">
      <div class="stat-card">
        <div class="stat-num">{{ mockResults.length }}</div>
        <div class="stat-label">Documenti analizzati</div>
      </div>
      <div class="stat-card">
        <div class="stat-num" style="color: var(--orange-600)">{{ mockResults.length }}</div>
        <div class="stat-label">Classificazioni completate</div>
      </div>
      <div class="stat-card">
        <div class="stat-num">{{ avgConfidence }}%</div>
        <div class="stat-label">Confidence media</div>
      </div>
    </div>

    <!-- Result table -->
    <div class="result-table-wrap">
      <div class="table-header">
        <span>Documento</span>
        <span>Confidence</span>
        <span>Stato</span>
      </div>
      <div class="table-body">
        <div class="table-row" v-for="(r, i) in mockResults" :key="i">
          <div class="td-file">
            <div class="file-icon-sm">📄</div>
            <div>
              <div class="file-name-sm">{{ r.file }}</div>
              <div class="file-meta-sm">{{ r.tokens.toLocaleString() }} token · {{ r.entities }} entità</div>
            </div>
          </div>
          <div class="td-conf">
            <div class="conf-bar-mini">
              <div class="conf-fill-mini" :style="{ width: (parseFloat(r.confidence) * 100) + '%' }" />
            </div>
            <span class="conf-num" :style="{ color: confidenceColor(r.confidence) }">
              {{ (parseFloat(r.confidence) * 100).toFixed(0) }}%
            </span>
          </div>
          <div class="td-status">
            <span class="status-badge">✓ {{ r.status }}</span>
          </div>
        </div>
      </div>
    </div>

    <!-- Action buttons -->
    <div class="result-actions">
      <button class="btn btn-secondary result-btn" @click="showModal = true">
        <svg viewBox="0 0 20 20" fill="none" width="18" height="18">
          <rect x="3" y="3" width="14" height="14" rx="2" stroke="currentColor" stroke-width="1.5"/>
          <path d="M7 8h6M7 11h4" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
        </svg>
        Visualizza risultati
      </button>
      <button class="btn btn-primary result-btn" @click="downloadCSV">
        <svg viewBox="0 0 20 20" fill="none" width="18" height="18">
          <path d="M10 3v10m-4-4l4 4 4-4" stroke="currentColor" stroke-width="1.6"
            stroke-linecap="round" stroke-linejoin="round"/>
          <path d="M4 16h12" stroke="currentColor" stroke-width="1.6" stroke-linecap="round"/>
        </svg>
        Scarica risultati CSV
      </button>
    </div>

  </div>

  <!-- ── Results Modal ───────────────────────────────── -->
  <Teleport to="body">
    <Transition name="modal-fade">
      <div class="modal-backdrop" v-if="showModal" @click.self="showModal = false">
        <div class="modal-box">
          <div class="modal-top">
            <div>
              <h2 class="modal-title">Risultati dettagliati</h2>
              <p class="modal-sub">Pipeline DiInfo UniPg — {{ new Date().toLocaleDateString('it-IT') }}</p>
            </div>
            <button class="modal-close" @click="showModal = false">
              <svg viewBox="0 0 20 20" fill="none" width="18" height="18">
                <path d="M4 4l12 12M16 4L4 16" stroke="currentColor" stroke-width="1.8" stroke-linecap="round"/>
              </svg>
            </button>
          </div>

          <div class="modal-stats">
            <div v-for="(r, i) in mockResults" :key="i" class="modal-doc-card">
              <div class="mdc-top">
                <span class="mdc-name">{{ r.file }}</span>
                <span class="status-badge">✓ Classificato</span>
              </div>
              <div class="mdc-bars">
                <div class="mdc-bar-row">
                  <span>Confidence</span>
                  <div class="mdc-track"><div class="mdc-fill orange" :style="{ width: (parseFloat(r.confidence)*100)+'%' }" /></div>
                  <span class="mdc-val">{{ (parseFloat(r.confidence)*100).toFixed(0) }}%</span>
                </div>
                <div class="mdc-bar-row">
                  <span>Token processati</span>
                  <div class="mdc-track"><div class="mdc-fill blue" :style="{ width: Math.min(r.tokens/3200*100,100)+'%' }" /></div>
                  <span class="mdc-val">{{ r.tokens.toLocaleString() }}</span>
                </div>
                <div class="mdc-bar-row">
                  <span>Entità estratte</span>
                  <div class="mdc-track"><div class="mdc-fill blue" :style="{ width: Math.min(r.entities/52*100,100)+'%' }" /></div>
                  <span class="mdc-val">{{ r.entities }}</span>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </Transition>
  </Teleport>
</template>

<style scoped>
.results-wrap { display: flex; flex-direction: column; gap: 20px; }

.stats-row {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 12px;
}
.stat-card {
  background: var(--surface);
  border: 1.5px solid var(--border);
  border-radius: var(--radius-md);
  padding: 18px 20px;
  text-align: center;
  box-shadow: var(--shadow-sm);
}
.stat-num {
  font-family: var(--font-head);
  font-size: 26px;
  font-weight: 700;
  color: var(--blue-700);
  line-height: 1;
  margin-bottom: 4px;
}
.stat-label { font-size: 12px; color: var(--text-3); line-height: 1.3; }

.result-table-wrap {
  background: var(--surface);
  border: 1.5px solid var(--border);
  border-radius: var(--radius-lg);
  overflow: hidden;
  box-shadow: var(--shadow-sm);
}
.table-header {
  display: grid;
  grid-template-columns: 3fr 1fr 1fr;
  gap: 12px;
  padding: 12px 20px;
  background: var(--bg);
  border-bottom: 1px solid var(--border);
  font-size: 11px;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  color: var(--text-3);
}
.table-body { display: flex; flex-direction: column; }
.table-row {
  display: grid;
  grid-template-columns: 3fr 1fr 1fr;
  gap: 12px;
  padding: 14px 20px;
  align-items: center;
  border-bottom: 1px solid var(--border);
  transition: background 0.15s;
}
.table-row:last-child { border-bottom: none; }
.table-row:hover { background: var(--bg); }

.td-file { display: flex; align-items: center; gap: 10px; }
.file-icon-sm { font-size: 18px; flex-shrink: 0; }
.file-name-sm { font-size: 13px; font-weight: 500; color: var(--text-1); }
.file-meta-sm { font-size: 11px; color: var(--text-3); margin-top: 1px; }

.td-conf { display: flex; align-items: center; gap: 8px; }
.conf-bar-mini {
  flex: 1;
  height: 4px;
  background: var(--blue-100);
  border-radius: 99px;
  overflow: hidden;
}
.conf-fill-mini {
  height: 100%;
  background: var(--orange-500);
  border-radius: 99px;
}
.conf-num { font-size: 12px; font-weight: 600; min-width: 36px; text-align: right; }

.status-badge {
  font-size: 11px;
  font-weight: 600;
  color: #15803d;
  background: #dcfce7;
  padding: 3px 9px;
  border-radius: 99px;
  white-space: nowrap;
}

.result-actions { display: flex; gap: 12px; }
.result-btn { font-size: 14px; padding: 12px 24px; }

.modal-backdrop {
  position: fixed;
  inset: 0;
  background: rgba(7, 18, 46, 0.55);
  backdrop-filter: blur(4px);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 100;
  padding: 24px;
}
.modal-box {
  background: var(--surface);
  border-radius: var(--radius-xl);
  width: 100%;
  max-width: 680px;
  max-height: 85vh;
  overflow-y: auto;
  display: flex;
  flex-direction: column;
  box-shadow: var(--shadow-xl);
}
.modal-top {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  padding: 28px 28px 20px;
  border-bottom: 1px solid var(--border);
  position: sticky;
  top: 0;
  background: var(--surface);
  z-index: 1;
}
.modal-title {
  font-family: var(--font-head);
  font-size: 20px;
  font-weight: 700;
  color: var(--blue-900);
}
.modal-sub { font-size: 13px; color: var(--text-3); margin-top: 2px; }
.modal-close {
  background: var(--bg);
  border: 1px solid var(--border);
  border-radius: 8px;
  padding: 8px;
  cursor: pointer;
  color: var(--text-2);
  display: flex;
  transition: all 0.15s;
}
.modal-close:hover { background: #FFF0F0; color: #e03535; }

.modal-stats { padding: 20px 28px; display: flex; flex-direction: column; gap: 16px; }
.modal-doc-card {
  background: var(--bg);
  border-radius: var(--radius-md);
  padding: 16px;
  border: 1px solid var(--border);
}
.mdc-top {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 12px;
}
.mdc-name { font-size: 13px; font-weight: 600; color: var(--text-1); }
.mdc-bars { display: flex; flex-direction: column; gap: 10px; }
.mdc-bar-row {
  display: flex;
  align-items: center;
  gap: 10px;
  font-size: 12px;
  color: var(--text-2);
}
.mdc-bar-row > span:first-child { min-width: 130px; }
.mdc-track {
  flex: 1;
  height: 5px;
  background: var(--border);
  border-radius: 99px;
  overflow: hidden;
}
.mdc-fill { height: 100%; border-radius: 99px; }
.mdc-fill.orange { background: var(--orange-500); }
.mdc-fill.blue   { background: var(--blue-500); }
.mdc-val { min-width: 50px; text-align: right; font-weight: 500; }

.modal-fade-enter-active, .modal-fade-leave-active { transition: all 0.25s ease; }
.modal-fade-enter-from, .modal-fade-leave-to { opacity: 0; }
.modal-fade-enter-from .modal-box { transform: translateY(20px) scale(0.97); }
</style>