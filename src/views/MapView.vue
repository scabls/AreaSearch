<template>
  <div class="map-view">
    <header class="map-header">
      <router-link to="/" class="local-name">{{ titleAddress }}</router-link>
      <form @submit.prevent="handleSearch" class="search-form">
        <input type="text" placeholder="搜索城市" v-model="keyword" class="search-input" />
      </form>
      <button @click="resetMap" class="reset-btn">复位地图</button>
    </header>
    <Map :geodata />
    <WeatherInfo :weather />
  </div>
</template>

<script setup>
import Map from '@/components/Map.vue'
import WeatherInfo from '@/components/WeatherInfo.vue'

import { ref, onMounted } from 'vue'
import { useRoute } from 'vue-router'
import { getWeather, getAdcode } from '@/api/weather'

const route = useRoute()
const { adcode } = route.params

const titleAddress = ref('')
const weather = ref({})
const geodata = ref({})
const keyword = ref('')

let initiaGeoData, initiaWeather

const handleSearch = async () => {
  try {
    const res = await getAdcode(keyword.value).then(res => res.geocodes[0])
    if (res.level !== '国家' && res.level !== '省' && res.level !== '市') return
    geodata.value = res
    try {
      const res2 = await getWeather(res.adcode).then(res => res.lives[0])
      weather.value = res2
    } catch (error) {}
  } catch (error) {}
  keyword.value = ''
}
const resetMap = () => {
  weather.value = initiaWeather
  geodata.value = initiaGeoData
}

onMounted(async () => {
  const weatherRes = await getWeather(adcode).then(res => res.lives[0])
  weather.value = weatherRes
  initiaWeather = weatherRes
  titleAddress.value = weatherRes.city
  const geodataRes = await getAdcode(weatherRes.city).then(res => res.geocodes[0])
  geodata.value = geodataRes
  initiaGeoData = geodataRes
})
</script>

<style lang="scss">
.map-view {
  position: fixed;
  inset: 0;
  .map-header {
    display: flex;
    gap: 2rem;
    padding: 0.5rem 2rem;
    background-color: #4264fb;
    .local-name {
      color: #fff;
      text-decoration: none;
      align-content: center;
    }
    .search-form {
      height: 2.5rem;
      .search-input {
        height: 100%;
        padding-left: 1rem;
        border-radius: 0.3rem;
      }
    }
    .reset-btn {
      background-color: #fff;
      padding-left: 1rem;
      padding-right: 1rem;
      border-radius: 0.3rem;
      cursor: pointer;
    }
  }
}
</style>
