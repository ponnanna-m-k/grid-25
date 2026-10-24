<script setup>
defineProps({
  cars: {
    type: Array,
    required: true,
  },
  filteredCars: {
    type: Array,
    required: true,
  },
  series: {
    type: Array,
    required: true,
  },
  searchQuery: {
    type: String,
    required: true,
  },
  selectedSeries: {
    type: String,
    required: true,
  },
  filterOnlyTh: {
    type: Boolean,
    required: true,
  },
})

defineEmits([
  'update:search-query',
  'update:selected-series',
  'reset-filters',
  'select-car',
])
</script>

<template>
  <main id="collection" class="site-container collection-main mx-auto px-4 py-12 sm:px-6 lg:px-8">
    <div class="filter-bar mb-10 flex flex-col items-stretch justify-between gap-4 rounded-2xl border p-2 md:flex-row md:items-center">
      <div class="relative flex-1">
        <input
          :value="searchQuery"
          type="search"
          placeholder="Search casting name, color, or year..."
          class="w-full rounded-xl border px-4 py-2.5 text-sm focus:outline-none"
          @input="$emit('update:search-query', $event.target.value)"
        />
      </div>

      <div class="flex items-center gap-3">
        <select
          :value="selectedSeries"
          class="rounded-xl border px-4 py-2.5 text-sm focus:outline-none"
          @change="$emit('update:selected-series', $event.target.value)"
        >
          <option value="">All Series ({{ cars.length }})</option>
          <option v-for="item in series" :key="item" :value="item">
            {{ item }}
          </option>
        </select>

        <button
          v-if="searchQuery || selectedSeries || filterOnlyTh"
          class="reset-button px-3 py-2 text-xs"
          type="button"
          @click="$emit('reset-filters')"
        >
          Reset
        </button>
      </div>
    </div>

    <div class="collection-heading">
      <h2>My collection</h2>
      <p>Showing {{ filteredCars.length }} of {{ cars.length }} cars</p>
    </div>

    <div v-if="filteredCars.length" class="car-grid grid grid-cols-1 gap-6 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4">
      <article
        v-for="car in filteredCars"
        :key="car.id"
        class="car-card group relative flex cursor-pointer flex-col justify-between overflow-hidden rounded-2xl border transition-all duration-300"
        tabindex="0"
        role="button"
        :aria-label="`Inspect ${car.name}`"
        @click="$emit('select-car', car)"
        @keydown.enter="$emit('select-car', car)"
        @keydown.space.prevent="$emit('select-car', car)"
      >
        <div class="car-image relative aspect-[4/3] w-full overflow-hidden">
          <img
            :src="car.image"
            :alt="car.name"
            class="h-full w-full object-contain transition-transform duration-500 group-hover:scale-105"
            loading="lazy"
          />
          <div class="absolute right-3 top-3 flex flex-col items-end gap-1.5">
            <span
              v-if="car.isTreasureHunt"
              class="rounded-md bg-gradient-to-r from-red-500 to-red-600 px-2.5 py-1 text-[11px] font-bold text-zinc-950 shadow-md"
            >
              🔥 {{ car.rarity }}
            </span>
            <span
              v-else-if="car.rarity === 'Premium'"
              class="rounded-md border border-zinc-700 bg-zinc-900/90 px-2.5 py-1 text-[11px] font-semibold text-zinc-200"
            >
              ⭐ Premium
            </span>
            <span
              v-else
              class="rounded-md border border-zinc-800 bg-zinc-950/80 px-2.5 py-1 text-[11px] font-medium text-zinc-300"
            >
              {{ car.series }}
            </span>
          </div>
        </div>

        <div class="car-details flex flex-1 flex-col justify-between p-5">
          <div>
            <div class="mb-1 flex items-center justify-between font-mono text-xs text-red-400">
              <span>{{ car.year }} RELEASE</span>
              <span class="text-zinc-500">#{{ car.id }}</span>
            </div>
            <h3 class="text-lg font-bold transition-colors group-hover:text-red-400">{{ car.name }}</h3>
            <p class="mt-1.5 text-xs">Color: {{ car.color }}</p>
          </div>
          <div class="mt-4 flex items-center justify-between border-t border-zinc-800/80 pt-3 text-xs">
            <span>{{ car.series }}</span>
            <span class="text-red-400">Inspect →</span>
          </div>
        </div>
      </article>
    </div>

    <div v-else class="empty-state rounded-3xl border border-dashed py-20 text-center">
      <p>No cars found matching filters.</p>
      <button class="mt-4 text-xs text-red-400 underline" type="button" @click="$emit('reset-filters')">
        Reset Filters
      </button>
    </div>
  </main>
</template>
