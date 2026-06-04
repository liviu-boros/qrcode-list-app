<script setup>
import { ref, computed, onMounted } from 'vue';
import Card from 'primevue/card';
import Button from 'primevue/button';
import SelectButton from 'primevue/selectbutton';
import InputText from 'primevue/inputtext';
import Message from 'primevue/message';
import InputGroup from 'primevue/inputgroup';
import InputGroupAddon from 'primevue/inputgroupaddon';
import Badge from 'primevue/badge';
import QrCode from './components/QrCode.vue';

// Generate a random unique ID for lists
const generateId = () => Date.now() + Math.random().toString(36).substring(2, 9);

// Dark/Light Theme state
const isDark = ref(false);

const toggleTheme = () => {
  isDark.value = !isDark.value;
  updateTheme();
};

const updateTheme = () => {
  const html = document.documentElement;
  if (isDark.value) {
    html.classList.add('p-dark');
    html.classList.remove('p-light');
    localStorage.setItem('theme', 'dark');
  } else {
    html.classList.remove('p-dark');
    html.classList.add('p-light');
    localStorage.setItem('theme', 'light');
  }
};

// Focused section tracking for premium UI cross-linking
const focusedSectionId = ref(null);

// Configuration mode options
const modeOptions = [
  { label: 'Simple Mode', value: 'simple' },
  { label: 'ID Suffix (#)', value: 'suffix' }
];

// Multiplier options
const multiplierOptions = [
  { label: 'None', value: 0 },
  { label: '+1', value: 1 },
  { label: '+2', value: 2 },
  { label: '+3', value: 3 },
  { label: '+4', value: 4 },
  { label: '+5', value: 5 }
];

// Reactive list of generative sections
const sections = ref([
  {
    id: generateId(),
    mode: 'simple',
    text: '',
    multiplier: 0
  }
]);

// Helper to determine if a string is a clean integer
const isCleanInteger = (val) => {
  return /^\d+$/.test(val.trim());
};

// Add a new section with default simple mode settings
const addSection = () => {
  sections.value.push({
    id: generateId(),
    mode: 'simple',
    text: '',
    multiplier: 0
  });
};

// Remove a section by ID, making sure we handle empty lists gracefully
const removeSection = (id) => {
  sections.value = sections.value.filter(s => s.id !== id);
  if (sections.value.length === 0) {
    addSection(); // Automatically keep at least one active section
  }
};

// Reset to initial clean single section state
const clearAll = () => {
  sections.value = [
    {
      id: generateId(),
      mode: 'simple',
      text: '',
      multiplier: 0
    }
  ];
};

// Print utility wrapper
const printList = () => {
  window.print();
};

// Computed array of processed sections containing their generated QR code payloads
const processedSections = computed(() => {
  return sections.value.map((section, idx) => {
    const textTrimmed = section.text.trim();
    const isSuffix = section.mode === 'suffix';
    const isInteger = isCleanInteger(textTrimmed);
    
    // Multiplier is strict: only active in suffix mode when input is a valid integer
    const activeMultiplier = isSuffix && isInteger ? parseInt(section.multiplier || 0, 10) : 0;
    
    const payloads = [];
    let hasAlphaWarning = false;

    if (textTrimmed !== '') {
      if (isSuffix) {
        if (isInteger) {
          const startNum = parseInt(textTrimmed, 10);
          const padLength = textTrimmed.length;
          
          for (let i = 0; i <= activeMultiplier; i++) {
            const incrementedNum = startNum + i;
            // Retain original padding length for a professional sequential experience
            const numString = String(incrementedNum).padStart(padLength, '0');
            payloads.push(`#${numString}`);
          }
        } else {
          // Fallback logic for non-integer inputs in suffix mode
          payloads.push(`#${textTrimmed}`);
          if (parseInt(section.multiplier || 0, 10) > 0) {
            hasAlphaWarning = true;
          }
        }
      } else {
        // Simple Mode outputs the string raw
        payloads.push(textTrimmed);
      }
    }

    return {
      ...section,
      index: idx,
      displayName: `Section ${idx + 1}`,
      isInteger,
      activeMultiplier,
      payloads,
      hasAlphaWarning
    };
  });
});

// Load saved theme on startup
onMounted(() => {
  const savedTheme = localStorage.getItem('theme');
  if (savedTheme) {
    isDark.value = savedTheme === 'dark';
  } else {
    isDark.value = window.matchMedia('(prefers-color-scheme: dark)').matches;
  }
  updateTheme();
});
</script>

<template>
  <div class="app-wrapper">
    <!-- Header Area -->
    <header class="app-header">
      <div class="header-title-area">
        <div class="header-logo-container">
          <i class="pi pi-qrcode"></i>
        </div>
        <div>
          <h1 class="header-title">QR Code List Generator</h1>
          <p class="header-subtitle">Enterprise Real-Time Scanner Batch Utility</p>
        </div>
      </div>
      
      <div class="header-actions">
        <Button 
          icon="pi pi-plus" 
          label="Add Section" 
          severity="primary" 
          @click="addSection" 
          raised
          id="btn-add-section-top"
        />
        <Button 
          icon="pi pi-print" 
          label="Print Codes" 
          severity="success" 
          @click="printList" 
          outlined
          id="btn-print-list"
        />
        <Button 
          icon="pi pi-trash" 
          label="Clear All" 
          severity="danger" 
          @click="clearAll" 
          text
          id="btn-clear-all"
        />
        <Button 
          :icon="isDark ? 'pi pi-sun' : 'pi pi-moon'" 
          :aria-label="isDark ? 'Switch to Light Mode' : 'Switch to Dark Mode'"
          severity="secondary" 
          variant="text" 
          rounded 
          @click="toggleTheme" 
          id="btn-theme-toggle"
        />
      </div>
    </header>

    <!-- Main Workspace Grid -->
    <main class="sections-list" aria-label="QR Generator Sections">
      <TransitionGroup name="list">
        <div 
          v-for="(section, idx) in processedSections" 
          :key="section.id" 
          class="section-row"
          :id="'section-row-' + section.id"
        >
          <!-- Left Column: Form Settings -->
          <div class="input-pane">
            <Card 
              class="config-card"
              :class="{ 'card-focused': focusedSectionId === section.id }"
              :id="'config-card-' + section.id"
            >
              <template #content>
                <div class="card-header-row">
                  <h2 class="card-title" :id="'card-title-' + section.id">
                    <span class="card-title-index">{{ idx + 1 }}</span>
                  </h2>
                  <Button 
                    icon="pi pi-times" 
                    severity="danger" 
                    variant="text" 
                    rounded 
                    aria-label="Delete Section"
                    class="delete-btn"
                    @click="removeSection(section.id)"
                    :id="'btn-delete-section-' + section.id"
                  />
                </div>

                <!-- Input Mode Selection -->
                <div class="control-field">
                  <span class="control-label">
                    <i class="pi pi-cog"></i> Mode Selection
                  </span>
                  <SelectButton 
                    v-model="sections[idx].mode" 
                    :options="modeOptions" 
                    optionLabel="label" 
                    optionValue="value" 
                    :allowEmpty="false"
                    :id="'mode-selector-' + section.id"
                  />
                </div>

                <!-- Value / Text Input Group -->
                <div class="control-field">
                  <label :for="'input-text-' + section.id" class="control-label">
                    <i class="pi pi-pencil"></i> Payload Input
                  </label>
                  <InputGroup>
                    <InputGroupAddon 
                      v-if="sections[idx].mode === 'suffix'" 
                      class="addon-hash"
                      aria-hidden="true"
                    >
                      #
                    </InputGroupAddon>
                    <InputText 
                      :id="'input-text-' + section.id"
                      v-model="sections[idx].text"
                      :placeholder="sections[idx].mode === 'suffix' ? 'Enter starting base integer (e.g. 300)' : 'Enter text payload...'"
                      @focus="focusedSectionId = section.id"
                      @blur="focusedSectionId = null"
                      autocomplete="off"
                    />
                  </InputGroup>
                </div>

                <!-- Multiplier Selector -->
                <div class="control-field">
                  <span class="control-label" :class="{ 'text-muted': sections[idx].mode !== 'suffix' }">
                    <i class="pi pi-clone"></i> Auto-Increment Multiplier
                  </span>
                  <SelectButton 
                    v-model="sections[idx].multiplier" 
                    :options="multiplierOptions" 
                    optionLabel="label" 
                    optionValue="value" 
                    :disabled="sections[idx].mode !== 'suffix'"
                    :allowEmpty="false"
                    :id="'multiplier-' + section.id"
                  />
                  
                  <!-- Graceful Fallback Warning Message -->
                  <Message 
                    v-if="sections[idx].mode === 'suffix' && sections[idx].text.trim() !== '' && !isCleanInteger(sections[idx].text) && sections[idx].multiplier > 0" 
                    severity="warn" 
                    size="small"
                    variant="simple"
                    class="mt-2"
                  >
                    Alphanumeric text detected. Multiplier disabled, generating single code.
                  </Message>
                </div>
              </template>
            </Card>
          </div>

          <!-- Right Column: Visual QR Viewport -->
          <div class="output-pane">
            <Card 
              class="output-card"
              :class="{ 'card-highlighted': focusedSectionId === section.id }"
              :id="'output-card-' + section.id"
            >
              <template #content>
                <div class="output-header-row">
                  <div class="output-title" :id="'output-title-' + section.id">
                    <span class="card-title-index">{{ section.index + 1 }}</span>
                  </div>
                  <Badge 
                    :value="section.payloads.length === 0 ? 'Empty' : (section.payloads.length + ' Code' + (section.payloads.length > 1 ? 's' : ''))" 
                    :severity="section.payloads.length === 0 ? 'secondary' : 'success'"
                    class="output-badge-count"
                  />
                </div>

                <!-- Stack of QR code items -->
                <div class="qr-feed-container">
                  <!-- If section text is empty, show the 192x192 placeholder -->
                  <div v-if="section.payloads.length === 0" class="qr-item-wrapper">
                    <QrCode value="" />
                  </div>
                  
                  <!-- Otherwise loop through the generated payloads -->
                  <template v-else>
                    <div 
                      v-for="(payload, pIdx) in section.payloads" 
                      :key="payload + '-' + pIdx" 
                      class="qr-item-wrapper"
                    >
                      <QrCode :value="payload" />
                      <span class="qr-label-tag" :title="payload">{{ payload }}</span>
                    </div>
                  </template>
                </div>
              </template>
            </Card>
          </div>
        </div>
      </TransitionGroup>

      <!-- Bottom Add Section Button Row -->
      <div class="sections-footer-actions">
        <div class="input-pane">
          <Button 
            icon="pi pi-plus" 
            label="Add New Section Row" 
            severity="primary" 
            outlined 
            class="w-full py-3" 
            @click="addSection"
            id="btn-add-section-bottom"
          />
        </div>
        <div class="output-pane">
          <!-- Spans empty to align button with input cards on left -->
        </div>
      </div>
    </main>
    
    <!-- Footer Context -->
    <footer class="text-center py-6 text-sm text-gray-500 footer-text">
      <p>QR Code List App &bull; Designed for fast industrial utility scanners &bull; Use Ctrl+P to print list</p>
    </footer>
  </div>
</template>

<style>
/* Additional component alignment tweaks */
.w-full {
  width: 100%;
}
.py-3 {
  padding-top: 0.75rem;
  padding-bottom: 0.75rem;
}
.mt-2 {
  margin-top: 0.5rem;
}
.text-muted {
  opacity: 0.5;
}

/* Focused card indicator (Left side) */
.config-card.card-focused {
  border-color: var(--p-primary-color, #10b981) !important;
  box-shadow: 0 4px 20px rgba(16, 185, 129, 0.08) !important;
}

/* Highlighted output card indicator (Right side) when focusing inputs */
.output-card.card-highlighted {
  border-color: var(--p-primary-color, #10b981) !important;
  box-shadow: 0 4px 20px rgba(16, 185, 129, 0.08) !important;
}
</style>
