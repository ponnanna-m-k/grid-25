<script setup>
defineProps({
  darkTheme: {
    type: Boolean,
    required: true,
  },
  filterOnlyTh: {
    type: Boolean,
    required: true,
  },
})

defineEmits(['toggle-theme', 'toggle-treasure-hunts'])
</script>

<template>
  <header class="site-header sticky top-0 z-40 border-b bg-zinc-950/70 backdrop-blur-md">
    <div class="site-container mx-auto flex h-16 items-center justify-between px-4 sm:px-6 lg:px-8">
      <div class="flex items-center gap-3">
        <img class="brand-mark h-10 w-10 object-contain" src="/grid25-logo.png" alt="" />
        <h1 class="flex items-center gap-2 text-lg font-bold tracking-tight">
          <span class="brand-name">GRID 25</span>
          <span class="brand-scale rounded-full border border-orange-500/20 bg-orange-500/10 px-2 py-0.5 font-mono text-xs text-orange-400">1:64</span>
        </h1>
      </div>

      <div class="flex items-center gap-2 sm:gap-3">
        <button
          class="theme-toggle flex items-center gap-1.5 rounded-lg border px-3.5 py-1.5 text-xs font-semibold transition-all"
          type="button"
          :aria-pressed="darkTheme"
          @click="$emit('toggle-theme')"
        >
          <span aria-hidden="true">{{ darkTheme ? '☀️' : '🌙' }}</span>
        </button>
      </div>
    </div>
  </header>
</template>
