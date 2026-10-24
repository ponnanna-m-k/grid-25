<script setup>
import { onBeforeUnmount, ref, watch } from 'vue'

const props = defineProps({
  car: {
    type: Object,
    default: null,
  },
})

defineEmits(['close'])

const detailsExpanded = ref(false)
const detailsStatus = ref('idle')
const productDescription = ref('')
const detailsError = ref('')
let requestController

watch(() => props.car?.id, () => {
  requestController?.abort()
  detailsExpanded.value = false
  detailsStatus.value = 'idle'
  productDescription.value = ''
  detailsError.value = ''
})

onBeforeUnmount(() => requestController?.abort())

function extractReaderDescription(readerResponse) {
  const markdown = readerResponse.split(/Markdown Content:\s*/i)[1] || readerResponse
  const description = markdown.match(
    /(?:^|\n)\s*\[?Description\]?(?:\([^)]+\))?(?:\s+Details\s+Our Guarantee)?\s*\n+([\s\S]*?)(?=\n\s*(?:Company:|Brand:|Item Type:|Series:|Release Date:|UPC:|Stock #:|Our Guarantees? to You|Close×)\b|$)/i,
  )?.[1]

  if (!description) return ''

  return description
    .split('\n')
    .map((line) => line
      .replace(/!\[([^\]]*)\]\([^)]*\)/g, '$1')
      .replace(/\[([^\]]+)\]\([^)]*\)/g, '$1')
      .replace(/[`*_>#]/g, '')
      .trim())
    .filter((line) => line && !/^[-|]+$/.test(line))
    .join('\n')
    .slice(0, 2000)
    .trim()
}

async function toggleProductDetails() {
  if (detailsStatus.value === 'loaded') {
    detailsExpanded.value = !detailsExpanded.value
    return
  }

  if (detailsStatus.value === 'loading' || !props.car?.detailsSourceUrl) return

  try {
    if (!['http:', 'https:'].includes(new URL(props.car.detailsSourceUrl).protocol)) {
      throw new TypeError('Product details URL must use HTTP or HTTPS')
    }
  } catch {
    detailsStatus.value = 'error'
    return
  }

  requestController = new AbortController()
  detailsStatus.value = 'loading'
  detailsError.value = ''

  try {
    const response = await fetch(props.car.detailsSourceUrl, {
      signal: requestController.signal,
    })
    if (response.status === 404) {
      detailsStatus.value = 'unavailable'
      return
    }
    if (!response.ok) {
      throw new Error(`Product request failed (${response.status})`)
    }

    const responseBody = await response.text()
    let product
    try {
      product = JSON.parse(responseBody)
    } catch {
      product = null
    }

    productDescription.value = product && typeof product === 'object'
      ? typeof product.description === 'string'
        ? new DOMParser().parseFromString(product.description, 'text/html').body.textContent?.replace(/\s+/g, ' ').trim() || ''
        : ''
      : extractReaderDescription(responseBody)
    detailsStatus.value = productDescription.value ? 'loaded' : 'unavailable'
    detailsExpanded.value = detailsStatus.value === 'loaded'
  } catch (error) {
    if (error.name === 'AbortError') return
    detailsError.value = error.message
    detailsStatus.value = 'error'
  }
}
</script>

<template>
  <div
    v-if="car"
    class="car-modal fixed inset-0 z-50 flex items-center justify-center p-4 backdrop-blur-md"
    @click.self="$emit('close')"
  >
    <section
      class="car-modal-panel relative w-full max-w-2xl overflow-hidden rounded-3xl border shadow-2xl"
      role="dialog"
      aria-modal="true"
      aria-labelledby="car-modal-title"
    >
      <div class="car-modal-accent" aria-hidden="true"></div>
      <button
        class="car-modal-close absolute right-4 top-4 z-10 flex h-10 w-10 items-center justify-center rounded-full border transition"
        type="button"
        aria-label="Close details"
        @click="$emit('close')"
      >
        ✕
      </button>
      <div class="car-modal-visual relative flex items-center justify-center">
        <span class="car-modal-number absolute left-6 top-6 font-mono text-xs tracking-[0.2em]">
          COLLECTION / {{ String(car.id).padStart(2, '0') }}
        </span>
        <img :src="car.image" :alt="car.name" class="car-modal-image aspect-video w-full object-contain p-10 sm:p-14" />
        <span class="car-modal-stamp absolute bottom-5 right-6 rounded-full border px-3 py-1 font-mono text-xs">
          1:64 SCALE
        </span>
      </div>
      <div class="car-modal-content p-6 sm:p-8">
        <div class="mb-4 flex flex-wrap items-center gap-2">
          <span class="car-modal-chip car-modal-year rounded-full px-3 py-1 font-mono text-xs">{{ car.year }}</span>
          <span class="car-modal-chip rounded-full px-3 py-1 text-xs">{{ car.series }}</span>
          <span class="car-modal-chip rounded-full px-3 py-1 text-xs">{{ car.rarity }}</span>
        </div>
        <h2 id="car-modal-title" class="car-modal-title text-2xl font-bold leading-tight sm:text-3xl">{{ car.name }}</h2>
        <p class="car-modal-notes mt-3 text-sm leading-relaxed">{{ car.notes }}</p>
        <div class="car-modal-specs mt-6 grid grid-cols-2 gap-3">
          <div class="car-modal-spec rounded-2xl border p-4">
            <span class="car-modal-label block font-mono text-[10px] tracking-[0.16em]">BODY COLOR</span>
            <span class="car-modal-value mt-1 block text-sm font-semibold">{{ car.color }}</span>
          </div>
          <div class="car-modal-spec rounded-2xl border p-4">
            <span class="car-modal-label block font-mono text-[10px] tracking-[0.16em]">SERIES</span>
            <span class="car-modal-value mt-1 block text-sm font-semibold">{{ car.series }}</span>
          </div>
        </div>
        <button
          v-if="car.detailsSourceUrl"
          class="car-modal-source mt-6 inline-flex items-center gap-2 text-sm font-semibold transition-colors disabled:cursor-wait disabled:opacity-60"
          type="button"
          :disabled="detailsStatus === 'loading'"
          @click="toggleProductDetails"
        >
          {{ detailsStatus === 'loaded' && detailsExpanded ? 'See less' : detailsStatus === 'loading' ? 'Loading details…' : 'See more' }}
        </button>
        <p v-if="detailsStatus === 'loading'" class="car-modal-notes mt-3 text-sm" role="status">
          Fetching casting details…
        </p>
        <div
          v-if="detailsExpanded && detailsStatus === 'loaded'"
          class="car-modal-fetched-details mt-3 whitespace-pre-line text-sm leading-relaxed"
          role="region"
          aria-label="Fetched casting details"
          aria-live="polite"
        >
          <span class="car-modal-description-label">Description</span>
          <p class="mt-2">{{ productDescription }}</p>
        </div>
        <p v-if="detailsStatus === 'error'" class="car-modal-error mt-3 text-sm" role="status">
          Could not load product details: {{ detailsError }}
        </p>
        <p v-if="detailsStatus === 'unavailable'" class="car-modal-notes mt-3 text-sm" role="status">
          Additional details are not available for this source.
        </p>
      </div>
    </section>
  </div>
</template>
