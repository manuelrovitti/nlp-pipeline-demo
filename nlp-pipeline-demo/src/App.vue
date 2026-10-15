<script setup>
import { ref } from 'vue'
import FileDropZone from './components/FileDropZone.vue'
import PipelineLoader from './components/PipelineLoader.vue'
import AnnotationReview from './components/AnnotationReview.vue'
import ResultsPanel from './components/ResultsPanel.vue'
import ToastNotif from './components/ToastNotif.vue'

// ─── State Machine ───────────────────────────────────────────────
// 0 = upload  |  1 = loading NLP  |  2 = annotate
// 3 = loading classify  |  4 = results
const step = ref(0)

const uploadedFiles = ref([])
const selectedLabels = ref([])
const toast = ref({ show: false, message: '', type: 'success' })

function showToast(message, type = 'success') {
  toast.value = { show: true, message, type }
  setTimeout(() => { toast.value.show = false }, 5500)
}

// ─── Transitions ─────────────────────────────────────────────────
function startAnalysis() {
  if (!uploadedFiles.value.length) return
  step.value = 1
}

function onNLPDone() {
  step.value = 2
  showToast('Analisi completata. Rivedi le categorie rilevate prima di procedere.')
}

function startClassification() {
  if (!selectedLabels.value.length) return
  step.value = 3
}

function onClassificationDone() {
  step.value = 4
  showToast('Classificazione completata. I risultati sono pronti.')
}

// ─── Pipeline Steps Metadata ─────────────────────────────────────
const pipelineSteps = [
  { label: 'Caricamento' },
  { label: 'Analisi' },
  { label: 'Revisione' },
  { label: 'Classificazione' },
  { label: 'Risultati' },
]
</script>

<template>
  <div class="app-shell">

    <!-- ── Sidebar ─────────────────────────────────────── -->
    <aside class="sidebar">
      <div class="sidebar-brand">
        <div class="brand-mark">
          <span>Di</span><span style="color: var(--blue-100); font-size:11px;">Info</span>
        </div>
        <div class="brand-text">
          <div class="brand-title">Dipartimento di Informatica</div>
          <div class="brand-sub">Università degli Studi di Perugia</div>
        </div>
      </div>

      <nav class="pipeline-nav">
        <div class="pipeline-label">Pipeline di Analisi</div>
        <div class="pipeline-steps">
          <div
            v-for="(s, i) in pipelineSteps"
            :key="i"
            class="pipeline-step"
            :class="{
              'is-done':   step > i,
              'is-active': step === i,
              'is-future': step < i,
            }"
          >
            <div class="step-track">
              <div class="step-dot">
                <svg v-if="step > i" viewBox="0 0 14 14" fill="none" width="14" height="14">
                  <path d="M2.5 7l3 3 5.5-5.5" stroke="currentColor" stroke-width="1.8"
                    stroke-linecap="round" stroke-linejoin="round"/>
                </svg>
                <span v-else>{{ i + 1 }}</span>
              </div>
              <div class="step-connector" v-if="i < pipelineSteps.length - 1" />
            </div>
            <div class="step-info">
              <div class="step-num">Fase {{ i + 1 }}</div>
              <div class="step-label">{{ s.label }}</div>
            </div>
          </div>
        </div>
      </nav>

      <div class="sidebar-footer">Pipeline Research Demo &middot; DiInfo UniPg</div>
    </aside>

    <!-- ── Main Content ────────────────────────────────── -->
    <main class="main-area">
      <Transition name="step-fade" mode="out-in">

        <!-- STEP 0: Upload -->
        <section v-if="step === 0" key="upload" class="step-section">
          <div class="step-header">
            <div class="step-eyebrow">Fase 1 di 5</div>
            <h1>Carica i documenti</h1>
            <p>
              Trascina uno o più file nell'area sottostante, oppure clicca per sfogliare.
              La demo supporta qualsiasi tipo di file di testo (PDF, TXT, DOCX).
            </p>
          </div>
          <FileDropZone @files-changed="uploadedFiles = $event" />
          <div class="step-actions">
            <button
              class="btn btn-primary"
              :disabled="!uploadedFiles.length"
              @click="startAnalysis"
            >
              <svg viewBox="0 0 20 20" fill="none" width="18" height="18">
                <circle cx="10" cy="10" r="8" stroke="currentColor" stroke-width="1.5"/>
                <path d="M7 10l2 2 4-4" stroke="currentColor" stroke-width="1.5"
                  stroke-linecap="round" stroke-linejoin="round"/>
              </svg>
              Avvia Analisi
            </button>
            <span v-if="!uploadedFiles.length" class="hint">
              Aggiungi almeno un documento per procedere
            </span>
          </div>
        </section>

        <!-- STEP 1: Loading NLP -->
        <section v-else-if="step === 1" key="loading-nlp" class="step-section">
          <div class="step-header">
            <div class="step-eyebrow">Fase 2 di 5</div>
            <h1>Analisi in corso…</h1>
            <p>
              Il modello PaddleOCR-VL sta elaborando i documenti caricati.
              L'operazione viene eseguita localmente e può richiedere alcuni secondi.
            </p>
          </div>
          <PipelineLoader
            model-name="Modello PaddleOCR-VL — Estrazione Informazioni"
            :phases="[
              'Rendering Documenti',
              'Analisi morfologica delle singole pagine',
              'Estrazione relazioni semantiche',
              'Generazione File JSON e MD',
            ]"
            :duration="7000"
            @done="onNLPDone"
          />
        </section>

        <!-- STEP 2: Annotation Review -->
        <section v-else-if="step === 2" key="annotate" class="step-section">
          <div class="step-header">
            <div class="step-eyebrow">Fase 3 di 5</div>
            <h1>Scelta BioAssay</h1>
            <p>
              Il modello ha rilevato le seguenti categorie tematiche nei tuoi documenti.
              Seleziona quelle da includere nella classificazione finale.
            </p>
          </div>
          <AnnotationReview
            :file-count="uploadedFiles.length"
            @selection-changed="selectedLabels = $event"
          />
          <div class="step-actions">
            <button
              class="btn btn-primary"
              :disabled="!selectedLabels.length"
              @click="startClassification"
            >
              <svg viewBox="0 0 20 20" fill="none" width="18" height="18">
                <path d="M4 6h12M4 10h8M4 14h5" stroke="currentColor" stroke-width="1.5"
                  stroke-linecap="round"/>
              </svg>
              Avvia Classificazione
            </button>
            <span v-if="!selectedLabels.length" class="hint">
              Seleziona almeno una categoria
            </span>
            <span v-else class="hint" style="color: var(--orange-600); font-weight: 500;">
              {{ selectedLabels.length }} categorie selezionate
            </span>
          </div>
        </section>

        <!-- STEP 3: Loading Classification -->
        <section v-else-if="step === 3" key="loading-classify" class="step-section">
          <div class="step-header">
            <div class="step-eyebrow">Fase 4 di 5</div>
            <h1>Classificazione in corso…</h1>
            <p>
              Il modello di classificazione sta assegnando le categorie selezionate
              ai documenti e calcolando i confidence score.
            </p>
          </div>
          <PipelineLoader
            model-name="Modello Muse-Glimmer:30B — Classificazione Documenti"
            :phases="[
              'Acquisizione input',
              'Acquisizione Experiment Section',
              'BioAssay Label',
              'Citazione estratta',
            ]"
            :duration="6000"
            @done="onClassificationDone"
          />
        </section>

        <!-- STEP 4: Results -->
        <section v-else-if="step === 4" key="results" class="step-section">
          <div class="step-header">
            <div class="step-eyebrow">Fase 5 di 5</div>
            <h1>Risultati disponibili</h1>
            <p>
              La Pipeline ha completato l'analisi e la classificazione.
              Visualizza i risultati nell'interfaccia oppure scarica il report completo.
            </p>
          </div>
          <ResultsPanel
            :files="uploadedFiles"
            :selected-labels="selectedLabels"
          />
        </section>

      </Transition>
    </main>

    <!-- ── Toast Notification ───────────────────────── -->
    <ToastNotif :show="toast.show" :message="toast.message" />

  </div>
</template>
