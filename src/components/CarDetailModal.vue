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
const productDetails = ref('')
let requestController

watch(() => props.car?.id, () => {
  requestController?.abort()
  detailsExpanded.value = false
  detailsStatus.value = 'idle'
  productDetails.value = ''
})

onBeforeUnmount(() => requestController?.abort())

async function toggleProductDetails() {
  if (detailsStatus.value === 'loaded') {
    detailsExpanded.value = !detailsExpanded.value
    return
  }

  if (detailsStatus.value === 'loading' || !props.car?.imageSource) return

  let productApiUrl
  try {
    const sourceUrl = new URL(props.car.imageSource)
    if (!sourceUrl.pathname.includes('/products/')) {
      detailsStatus.value = 'unavailable'
      return
    }
    sourceUrl.pathname = `${sourceUrl.pathname.replace(/\/$/, '')}.js`
    productApiUrl = sourceUrl
  } catch {
    detailsStatus.value = 'error'
    return
  }

  requestController = new AbortController()
  detailsStatus.value = 'loading'

  try {
    const response = await fetch(productApiUrl, {
      headers: { Accept: 'application/json' },
      signal: requestController.signal,
    })
    if (response.status === 404) {
      detailsStatus.value = 'unavailable'
      return
    }
    if (!response.ok) {
      throw new Error(`Product request failed (${response.status})`)
    }

    const product = await response.json()
    const description = typeof product.description === 'string'
      ? new DOMParser().parseFromString(product.description, 'text/html').body.textContent.replace(/\s+/g, ' ').trim()
      : ''
    const details = [
      product.title,
      product.vendor && `Brand: ${product.vendor}`,
      product.type && `Series: ${product.type}`,
      description && `Description: ${description}`,
    ].filter(Boolean)
    productDetails.value = details.join('\n')
    detailsStatus.value = productDetails.value ? 'loaded' : 'unavailable'
    detailsExpanded.value = detailsStatus.value === 'loaded'
  } catch (error) {
    if (error.name === 'AbortError') return
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
          v-if="car.imageSource"
          class="car-modal-source mt-6 inline-flex items-center gap-2 text-sm font-semibold transition-colors disabled:cursor-wait disabled:opacity-60"
          type="button"
          :disabled="detailsStatus === 'loading'"
          @click="toggleProductDetails"
        >
          {{ detailsStatus === 'loaded' && detailsExpanded ? 'See less' : detailsStatus === 'loading' ? 'Loading details…' : 'See more' }}
        </button>
        <p v-if="detailsExpanded && detailsStatus === 'loaded'" class="car-modal-notes mt-3 whitespace-pre-line text-sm leading-relaxed">
          {{ productDetails }}
        </p>
        <p v-if="detailsStatus === 'error'" class="car-modal-error mt-3 text-sm" role="status">
          Could not load product details. Please try again.
        </p>
        <p v-if="detailsStatus === 'unavailable'" class="car-modal-notes mt-3 text-sm" role="status">
          Additional details are not available for this source.
        </p>
      </div>
    </section>
  </div>
</template>
