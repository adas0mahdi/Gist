<script setup lang="ts">
import { ref, onMounted, watch, onUnmounted, computed } from 'vue';

interface Recipient {
  email: string;
}

interface Props {
  recipients: Recipient[];
}

const props = defineProps<Props>();

const containerRef = ref<HTMLDivElement | null>(null);
const visibleRecipients = ref<string[]>([]);
const trimmedCount = ref(0);
const tooltipVisible = ref(false);

const calculateVisibleRecipients = () => {
  const container = containerRef.value;
  if (!container) return;

  const availableWidth = container.offsetWidth;
  let usedWidth = 0;
  let visible: string[] = [];

  const recipients = props.recipients.map((r) => r.email);
  const dummySpan = document.createElement('span');
  const containerStyles = getComputedStyle(container);

  dummySpan.style.visibility = 'hidden';
  dummySpan.style.whiteSpace = 'nowrap';
  dummySpan.style.font = containerStyles.font;
  document.body.appendChild(dummySpan);

  for (const recipient of recipients) {
    dummySpan.textContent = recipient + ', ';
    const recipientWidth = dummySpan.offsetWidth;

    if (usedWidth + recipientWidth <= availableWidth) {
      visible.push(recipient);
      usedWidth += recipientWidth;
    } else {
      break;
    }
  }

  document.body.removeChild(dummySpan);

  trimmedCount.value = recipients.length - visible.length;
  visibleRecipients.value = visible;
};

let resizeTimeout: number;
const onResize = () => {
  clearTimeout(resizeTimeout);
  resizeTimeout = window.setTimeout(calculateVisibleRecipients, 200);
};

onMounted(() => {
  window.addEventListener('resize', onResize);
  calculateVisibleRecipients();
});

watch(
  () => props.recipients,
  () => calculateVisibleRecipients()
);

onUnmounted(() => {
  window.removeEventListener('resize', onResize);
});

const tooltipContent = computed(() =>
  props.recipients.map((r) => r.email).join(', ')
);
</script>


<template>
    <div>
      <span>Debug: {{ recipients }}</span> <!-- Add this for debugging -->
      <span class="recipients-list">{{ visibleRecipients.join(', ') }}</span>
    </div>
  </template>
  

<style scoped>
.recipients-display {
  display: flex;
  align-items: center;
  width: 100%; /* Adjust based on your layout */
}

.recipients-list {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.recipients-tooltip {
  position: absolute;
  top: 8px;
  right: 8px;
  padding: 8px 16px;
  background-color: #666;
  color: #f0f0f0;
  border-radius: 8px;
  font-size: 14px;
  box-shadow: 0px 4px 6px rgba(0, 0, 0, 0.1);
  z-index: 1000;
}
</style>
