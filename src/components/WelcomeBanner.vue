<script setup>
defineProps({
  featuredCar: {
    type: Object,
    required: true,
  },
  totalCars: {
    type: Number,
    required: true,
  },
  treasureCount: {
    type: Number,
    required: true,
  },
  premiumCount: {
    type: Number,
    required: true,
  },
  seriesCount: {
    type: Number,
    required: true,
  },
})

defineEmits(['browse-hot-trucks'])
</script>

<template>
  <section class="hero-section overflow-hidden border-b">
    <div class="site-container relative z-10 mx-auto px-4 sm:px-6 lg:px-8">
      <div class="hero-layout">
        <div class="hero-copy">

          <h2 class="text-4xl font-extrabold tracking-tight sm:text-6xl lg:text-7xl">
            Small cars.
            <span>Big-time stories.</span>
          </h2>

          <p class="mt-6 max-w-2xl text-base leading-relaxed sm:text-lg">
            From classic muscle to trail-ready trucks, explore a handpicked garage of Hot Wheels favorites.
          </p>

          <div class="hero-actions">
            <a class="hero-action hero-action-primary" href="#collection">
              Explore the collection <span aria-hidden="true">↓</span>
            </a>
            <a
              class="hero-action hero-action-secondary"
              href="#collection"
              @click="$emit('browse-hot-trucks')"
            >
              Browse Hot Trucks <span aria-hidden="true">→</span>
            </a>
          </div>
        </div>

        <div class="hero-visual">
          <img :src="featuredCar.image" :alt="featuredCar.name" />
          <div class="hero-visual-caption">
            <span>Featured from the garage</span>
            <span>{{ featuredCar.name }}</span>
          </div>
        </div>
      </div>

      <div class="hero-stats grid grid-cols-2 gap-4 sm:grid-cols-4">
        <div class="stat-card rounded-2xl border p-4">
          <p class="font-mono text-2xl font-extrabold sm:text-3xl">{{ totalCars }}</p>
          <p class="mt-1 text-xs uppercase tracking-wider">Total Castings</p>
        </div>
        <div class="stat-card rounded-2xl border p-4">
          <p class="font-mono text-2xl font-extrabold sm:text-3xl">{{ treasureCount }}</p>
          <p class="mt-1 text-xs uppercase tracking-wider">TH / Super TH</p>
        </div>
        <div class="stat-card rounded-2xl border p-4">
          <p class="font-mono text-2xl font-extrabold sm:text-3xl">{{ premiumCount }}</p>
          <p class="mt-1 text-xs uppercase tracking-wider">Premiums</p>
        </div>
        <div class="stat-card rounded-2xl border p-4">
          <p class="font-mono text-2xl font-extrabold sm:text-3xl">{{ seriesCount }}</p>
          <p class="mt-1 text-xs uppercase tracking-wider">Unique Series</p>
        </div>
      </div>
    </div>
  </section>
</template>
