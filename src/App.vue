<script setup>
import { computed, ref } from 'vue'
import { carsData } from './data/cars.js'
import CarCollection from './components/CarCollection.vue'
import CarDetailModal from './components/CarDetailModal.vue'
import SiteHeader from './components/SiteHeader.vue'
import WelcomeBanner from './components/WelcomeBanner.vue'

const cars = ref(carsData)
const darkTheme = ref(true)
const filterOnlyTH = ref(false)
const searchQuery = ref('')
const selectedSeries = ref('')
const selectedCar = ref(null)

const uniqueSeries = computed(() => [...new Set(cars.value.map((car) => car.series))])
const countTreasures = computed(() => cars.value.filter((car) => car.isTreasureHunt).length)
const countPremiums = computed(() => cars.value.filter((car) => car.rarity === 'Premium').length)
const filteredCars = computed(() => {
  const query = searchQuery.value.toLowerCase()

  return cars.value.filter((car) => {
    const matchesSearch = car.name.toLowerCase().includes(query)
      || car.color.toLowerCase().includes(query)
      || car.year.toString().includes(query)
    const matchesSeries = !selectedSeries.value || car.series === selectedSeries.value
    const matchesTreasureHunt = !filterOnlyTH.value || car.isTreasureHunt

    return matchesSearch && matchesSeries && matchesTreasureHunt
  })
})

function resetFilters() {
  searchQuery.value = ''
  selectedSeries.value = ''
  filterOnlyTH.value = false
}

function browseHotTrucks() {
  searchQuery.value = ''
  selectedSeries.value = 'HW Hot Trucks'
  filterOnlyTH.value = false
}
</script>

<template>
  <div class="grid25-app min-h-screen" :class="{ 'dark-theme': darkTheme }">
    <SiteHeader
      :dark-theme="darkTheme"
      :filter-only-th="filterOnlyTH"
      @toggle-theme="darkTheme = !darkTheme"
      @toggle-treasure-hunts="filterOnlyTH = !filterOnlyTH"
    />

    <WelcomeBanner
      :featured-car="cars[0]"
      :total-cars="cars.length"
      :treasure-count="countTreasures"
      :premium-count="countPremiums"
      :series-count="uniqueSeries.length"
      @browse-hot-trucks="browseHotTrucks"
    />

    <CarCollection
      :cars="cars"
      :filtered-cars="filteredCars"
      :series="uniqueSeries"
      :search-query="searchQuery"
      :selected-series="selectedSeries"
      :filter-only-th="filterOnlyTH"
      @update:search-query="searchQuery = $event"
      @update:selected-series="selectedSeries = $event"
      @reset-filters="resetFilters"
      @select-car="selectedCar = $event"
    />

    <CarDetailModal :car="selectedCar" @close="selectedCar = null" />
  </div>
</template>
