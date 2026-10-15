<script setup>
import { ref } from 'vue'

const emit = defineEmits(['files-changed'])

const isDragging = ref(false)
const files = ref([])

function onDrop(e) {
  e.preventDefault()
  isDragging.value = false
  addFiles([...e.dataTransfer.files])
}

function onInputChange(e) {
  addFiles([...e.target.files])
  e.target.value = ''
}

function addFiles(newFiles) {
  const merged = [...files.value]
  for (const f of newFiles) {
    if (!merged.find(existing => existing.name === f.name)) {
      merged.push(f)
    }
  }
  files.value = merged
  emit('files-changed', merged)
}

function removeFile(index) {
  files.value.splice(index, 1)
  emit('files-changed', [...files.value])
}

function formatSize(bytes) {
  if (bytes < 1024) return bytes + ' B'
  if (bytes < 1024 * 1024) return (bytes / 1024).toFixed(1) + ' KB'
  return (bytes / (1024 * 1024)).toFixed(1) + ' MB'
}

function fileIcon(name) {
  const ext = name.split('.').pop().toLowerCase()
  if (ext === 'pdf') return '📄'
  if (['doc', 'docx'].includes(ext)) return '📝'
  if (['txt', 'md'].includes(ext)) return '🗒️'
  return '📎'
}
</script>

<template>
  <div class="dropzone-wrap">
    <!-- Drop Area -->
    <div
      class="drop-area"
      :class="{ 'is-dragging': isDragging }"
      @dragover.prevent="isDragging = true"
      @dragleave.prevent="isDragging = false"
      @drop="onDrop"
    >
      <div class="drop-icon">
        <svg viewBox="0 0 48 48" fill="none" width="48" height="48">
          <rect x="6" y="10" width="30" height="36" rx="4" stroke="currentColor" stroke-width="2"/>
          <path d="M14 10V7a3 3 0 013-3h16l9 9v25a3 3 0 01-3 3h-4" stroke="currentColor" stroke-width="2" stroke-linecap="round"/>
          <path d="M33 4v9h9" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
          <path d="M21 26v10M17 30l4-4 4 4" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
        </svg>
      </div>
      <div class="drop-label">
        <strong>Trascina i file qui</strong>
        <span>oppure</span>
        <label class="browse-btn">
          sfoglia i file
          <input
            type="file"
            multiple
            accept=".pdf,.txt,.docx,.doc,.md"
            style="display:none"
            @change="onInputChange"
          />
        </label>
      </div>
      <div class="drop-hint">PDF, TXT, DOCX — dimensione massima 50 MB per file</div>

      <!-- Drag overlay -->
      <div class="drag-overlay" v-if="isDragging">
        <div class="drag-overlay-text">Rilascia per aggiungere</div>
      </div>
    </div>

    <!-- File List -->
    <TransitionGroup name="file-list" tag="div" class="file-list" v-if="files.length">
      <div class="file-item" v-for="(file, i) in files" :key="file.name">
        <span class="file-icon">{{ fileIcon(file.name) }}</span>
        <div class="file-meta">
          <span class="file-name">{{ file.name }}</span>
          <span class="file-size">{{ formatSize(file.size) }}</span>
        </div>
        <button class="file-remove" @click="removeFile(i)" title="Rimuovi">
          <svg viewBox="0 0 16 16" fill="none" width="14" height="14">
            <path d="M3 3l10 10M13 3L3 13" stroke="currentColor" stroke-width="1.8" stroke-linecap="round"/>
          </svg>
        </button>
      </div>
    </TransitionGroup>

    <div v-if="files.length" class="file-count">
      {{ files.length }} {{ files.length === 1 ? 'documento caricato' : 'documenti caricati' }}
    </div>
  </div>
</template>

<style scoped>
.dropzone-wrap { display: flex; flex-direction: column; gap: 16px; }

.drop-area {
  position: relative;
  border: 2px dashed var(--border);
  border-radius: var(--radius-lg);
  background: var(--surface);
  padding: 48px 32px;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 12px;
  cursor: pointer;
  transition: border-color 0.2s, background 0.2s;
}
.drop-area:hover {
  border-color: var(--blue-300);
  background: var(--blue-50);
}
.drop-area.is-dragging {
  border-color: var(--orange-500);
  background: var(--orange-50);
}

.drop-icon { color: var(--text-3); }
.drop-area.is-dragging .drop-icon { color: var(--orange-500); }

.drop-label {
  display: flex;
  align-items: center;
  gap: 6px;
  font-size: 15px;
  color: var(--text-2);
}
.drop-label strong { color: var(--text-1); font-weight: 600; }
.browse-btn {
  color: var(--orange-600);
  font-weight: 600;
  cursor: pointer;
  text-decoration: underline;
  text-underline-offset: 2px;
}

.drop-hint {
  font-size: 12px;
  color: var(--text-3);
}

.drag-overlay {
  position: absolute;
  inset: 0;
  border-radius: calc(var(--radius-lg) - 2px);
  background: rgba(232, 118, 26, 0.08);
  display: flex;
  align-items: center;
  justify-content: center;
  pointer-events: none;
}
.drag-overlay-text {
  font-family: var(--font-head);
  font-size: 18px;
  font-weight: 600;
  color: var(--orange-600);
}

/* File list */
.file-list {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.file-item {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 12px 16px;
  background: var(--surface);
  border: 1.5px solid var(--border);
  border-radius: var(--radius-md);
  box-shadow: var(--shadow-sm);
}
.file-icon { font-size: 20px; flex-shrink: 0; }
.file-meta { flex: 1; min-width: 0; }
.file-name {
  display: block;
  font-size: 14px;
  font-weight: 500;
  color: var(--text-1);
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
.file-size {
  font-size: 12px;
  color: var(--text-3);
}
.file-remove {
  background: none;
  border: none;
  cursor: pointer;
  color: var(--text-3);
  padding: 4px;
  border-radius: 4px;
  display: flex;
  transition: color 0.15s, background 0.15s;
}
.file-remove:hover {
  color: #e03535;
  background: #FFF0F0;
}

.file-count {
  font-size: 13px;
  color: var(--text-3);
  text-align: right;
}

/* Transitions */
.file-list-enter-active, .file-list-leave-active { transition: all 0.22s ease; }
.file-list-enter-from { opacity: 0; transform: translateY(-6px); }
.file-list-leave-to   { opacity: 0; transform: translateX(12px); }
</style>
