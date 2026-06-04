<script setup>
import { ref, watch, onMounted } from 'vue';
import QRCode from 'qrcode';

const props = defineProps({
  value: {
    type: String,
    required: true
  },
  size: {
    type: Number,
    default: 192
  }
});

const qrDataUrl = ref('');
const error = ref(null);
const loading = ref(false);

const generateQr = async () => {
  if (!props.value) {
    qrDataUrl.value = '';
    return;
  }
  loading.value = true;
  error.value = null;
  try {
    // Generate QR code with high contrast, margin 2, and error correction level Q (high reliability)
    const url = await QRCode.toDataURL(props.value, {
      width: props.size,
      margin: 2,
      errorCorrectionLevel: 'Q',
      color: {
        dark: '#000000',
        light: '#FFFFFF'
      }
    });
    qrDataUrl.value = url;
  } catch (err) {
    console.error('QR Generation error:', err);
    error.value = err.message;
  } finally {
    loading.value = false;
  }
};

watch(() => props.value, generateQr);
onMounted(generateQr);
</script>

<template>
  <div class="qr-container-box">
    <div v-if="error" class="qr-error">
      <i class="pi pi-exclamation-triangle"></i>
      <span>Error generating QR</span>
    </div>
    <div v-else-if="!props.value" class="qr-placeholder">
      <i class="pi pi-qrcode placeholder-icon"></i>
      <span class="placeholder-text">Awaiting Input...</span>
    </div>
    <div v-else class="qr-image-wrapper">
      <img :src="qrDataUrl" :alt="props.value" class="qr-image" />
    </div>
  </div>
</template>

<style scoped>
.qr-container-box {
  width: 192px;
  height: 192px;
  min-width: 192px;
  max-width: 192px;
  min-height: 192px;
  max-height: 192px;
  background: var(--p-content-background, #ffffff);
  border: 2px dashed var(--p-content-border-color, #e2e8f0);
  border-radius: 12px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  overflow: hidden;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  position: relative;
  box-sizing: border-box;
}

.p-dark .qr-container-box {
  background: #18181b;
  border-color: #3f3f46;
}

.qr-placeholder {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  color: var(--p-text-muted-color, #71717a);
  user-select: none;
}

.placeholder-icon {
  font-size: 2.5rem;
  opacity: 0.4;
  animation: pulse 2.5s infinite ease-in-out;
}

.placeholder-text {
  font-size: 0.875rem;
  font-weight: 500;
  letter-spacing: 0.2px;
}

.qr-image-wrapper {
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  background: #ffffff; /* QR codes always need high contrast white background even in dark mode */
  padding: 8px;
  box-sizing: border-box;
}

.qr-image {
  width: 100%;
  height: 100%;
  object-fit: contain;
  display: block;
}

.qr-error {
  color: var(--p-red-500, #ef4444);
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  font-size: 0.75rem;
  text-align: center;
  padding: 12px;
}

@keyframes pulse {
  0%, 100% {
    opacity: 0.3;
    transform: scale(1);
  }
  50% {
    opacity: 0.6;
    transform: scale(1.04);
  }
}

/* Print optimizations */
@media print {
  .qr-container-box {
    border: 1px solid #000000 !important;
    box-shadow: none !important;
    transform: none !important;
    background: #ffffff !important;
    page-break-inside: avoid;
  }
  .placeholder-icon, .placeholder-text {
    color: #000000 !important;
  }
}
</style>
