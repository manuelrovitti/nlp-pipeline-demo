<script setup>
import { ref, computed, watch } from 'vue'

const props = defineProps({
  fileCount: { type: Number, default: 1 },
})
const emit = defineEmits(['selection-changed'])

// Mock categories detected by the NLP model
const categories = ref([
  {
    id: 1, label: 'Cell-based format',
    tag: 'C', color: 'blue', selected: true,
    description: 'Biological assays that use living cells to measure the effects of a substance on cellular function'
  },
  {
    id: 2, label: 'Target-based format',
    tag: 'T', color: 'orange', selected: true,
    description: 'Biological assays that target specific molecular entities to measure their effects'
  }
])

function toggle(cat) {
  cat.selected = !cat.selected
}

const selected = computed(() => categories.value.filter(c => c.selected))

watch(selected, val => {
  emit('selection-changed', val.map(c => c.label))
}, { immediate: true })

function pct(n) { return Math.round(n * 100) }
</script>

<template>
  <div class="annotation-wrap">

    <!-- Summary bar -->
    <div class="summary-bar">
      <div class="summary-item">
        <span class="summary-num">{{ fileCount }}</span>
        <span class="summary-label">{{ fileCount === 1 ? 'documento' : 'documenti' }} analizzati</span>
      </div>
      <div class="summary-divider" />
      <div class="summary-item">
        <span class="summary-num">{{ categories.length }}</span>
        <span class="summary-label">Possibili categorie</span>
      </div>
      <div class="summary-divider" />
      <div class="summary-item">
        <span class="summary-num" style="color: var(--orange-600)">{{ selected.length }}</span>
        <span class="summary-label">selezionate</span>
      </div>
    </div>

    <!-- Grid of category cards -->
    <div class="categories-grid">
      <button
        v-for="cat in categories"
        :key="cat.id"
        class="cat-card"
        :class="{ 'is-selected': cat.selected, [`color-${cat.color}`]: true }"
        @click="toggle(cat)"
      >
        <div class="cat-header">
          <div class="cat-tag" :class="`tag-${cat.color}`">{{ cat.tag }}</div>
          <div class="cat-check">
            <svg v-if="cat.selected" viewBox="0 0 16 16" fill="none" width="14" height="14">
              <circle cx="8" cy="8" r="7" fill="var(--orange-500)"/>
              <path d="M4.5 8l2.5 2.5 4.5-4.5" stroke="white" stroke-width="1.6"
                stroke-linecap="round" stroke-linejoin="round"/>
            </svg>
            <div v-else class="empty-check" />
          </div>
        </div>

        <div class="cat-label">{{ cat.label }}</div>
        <div class="cat-desc">{{ cat.description }}</div>

        
      </button>
    </div>

    <div class="annotation-hint">
      Clicca su una categoria per selezionarla o deselezionarla.
    </div>
  </div>
</template>

<style scoped>
.annotation-wrap { display: flex; flex-direction: column; gap: 20px; }

/* Summary bar */
.summary-bar {
  display: flex;
  align-items: center;
  gap: 20px;
  background: var(--surface);
  border: 1.5px solid var(--border);
  border-radius: var(--radius-md);
  padding: 16px 24px;
  box-shadow: var(--shadow-sm);
}
.summary-item { display: flex; align-items: baseline; gap: 6px; }
.summary-num {
  font-family: var(--font-head);
  font-size: 22px;
  font-weight: 700;
  color: var(--blue-700);
}
.summary-label { font-size: 13px; color: var(--text-2); }
.summary-divider {
  width: 1px;
  height: 24px;
  background: var(--border);
}

/* Grid */
.categories-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 12px;
}

.cat-card {
  all: unset;
  cursor: pointer;
  display: flex;
  flex-direction: column;
  gap: 8px;
  padding: 18px 20px;
  background: var(--surface);
  border: 1.5px solid var(--border);
  border-radius: var(--radius-md);
  transition: all 0.2s ease;
  box-shadow: var(--shadow-sm);
}
.cat-card:hover {
  border-color: var(--blue-300);
  transform: translateY(-2px);
  box-shadow: var(--shadow-md);
}
.cat-card.is-selected {
  border-color: var(--orange-500);
  background: var(--orange-50);
  box-shadow: 0 0 0 3px rgba(232, 118, 26, 0.12), var(--shadow-sm);
}

.cat-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.cat-tag {
  font-size: 10px;
  font-weight: 700;
  letter-spacing: 0.06em;
  padding: 3px 8px;
  border-radius: 4px;
}
.tag-blue  { background: var(--blue-100); color: var(--blue-700); }
.tag-orange { background: var(--orange-100); color: var(--orange-700); }
.tag-grey  { background: var(--bg); color: var(--text-2); border: 1px solid var(--border); }

.cat-check { color: var(--text-3); }
.empty-check {
  width: 18px;
  height: 18px;
  border-radius: 50%;
  border: 1.5px solid var(--border);
}

.cat-label {
  font-family: var(--font-head);
  font-size: 14px;
  font-weight: 600;
  color: var(--text-1);
  line-height: 1.3;
}
.cat-desc {
  font-size: 12px;
  color: var(--text-3);
  line-height: 1.45;
}

.cat-footer {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
  margin-top: 4px;
}
.conf-bar-wrap {
  display: flex;
  align-items: center;
  gap: 8px;
  flex: 1;
}
.conf-bar-track {
  flex: 1;
  height: 4px;
  background: var(--blue-100);
  border-radius: 99px;
  overflow: hidden;
}
.conf-bar-fill {
  height: 100%;
  background: var(--orange-500);
  border-radius: 99px;
  transition: width 0.6s ease;
}
.cat-card.is-selected .conf-bar-fill { background: var(--orange-600); }
.conf-value {
  font-size: 11px;
  font-weight: 600;
  color: var(--text-2);
  min-width: 30px;
  text-align: right;
}
.docs-badge {
  font-size: 11px;
  color: var(--text-3);
  white-space: nowrap;
}

.annotation-hint {
  font-size: 12px;
  color: var(--text-3);
  text-align: center;
  padding-top: 4px;
}
</style>
