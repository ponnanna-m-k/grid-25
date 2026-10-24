<script setup>
defineProps({
  car: {
    type: Object,
    default: null,
  },
})

defineEmits(['close'])
</script>

<template>
  <div
    v-if="car"
    class="car-modal fixed inset-0 z-50 flex items-center justify-center p-4 backdrop-blur-md"
    @click.self="$emit('close')"
  >
    <section
      class="car-modal-panel relative w-full max-w-lg overflow-hidden rounded-2xl border shadow-2xl"
      role="dialog"
      aria-modal="true"
      :aria-label="car.name"
    >
      <button
        class="absolute right-4 top-4 z-10 flex h-8 w-8 items-center justify-center rounded-full"
        type="button"
        aria-label="Close details"
        @click="$emit('close')"
      >
        ✕
      </button>
      <img :src="car.image" :alt="car.name" class="car-modal-image aspect-video w-full object-contain p-6" />
      <div class="p-6">
        <div class="mb-2 flex gap-2">
          <span class="rounded bg-red-500/20 px-2 py-0.5 text-xs text-red-400">{{ car.year }}</span>
          <span class="rounded bg-zinc-800 px-2 py-0.5 text-xs text-zinc-300">{{ car.rarity }}</span>
        </div>
        <h2 class="text-xl font-bold">{{ car.name }}</h2>
        <p class="mt-2 text-sm"><strong>Color:</strong> {{ car.color }}</p>
        <p class="mt-1 text-sm"><strong>Notes:</strong> {{ car.notes }}</p>
      </div>
    </section>
  </div>
</template>
