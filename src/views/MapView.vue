<template>
  <div class="map-view">
    <header class="map-header">
      <router-link to="/" class="local-name">{{ titleCityName }}</router-link>
      <form @submit.prevent="handleSearch">
        <input type="text" placeholder="搜索城市" v-model="keyword" />
      </form>
      <button @click="resetMap">复位</button>
    </header>
    <Map :city="weather.city" />
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

const titleCityName = ref('')
const weather = ref({})
const keyword = ref('')

const handleSearch = async () => {
  try {
    const res = await getAdcode(keyword.value).then(res => res.geocodes[0])
    if (res.level !== '全国' && res.level !== '省' && res.level !== '市') return
    try {
      const res2 = await getWeather(res.adcode).then(res => res.lives[0])
      weather.value = res2
    } catch (error) {}
  } catch (error) {}
  keyword.value = ''
}
const resetMap = async () => {
  const res = await getWeather(adcode).then(res => res.lives[0])
  weather.value = res
}

onMounted(async () => {
  const res = await getWeather(adcode).then(res => res.lives[0])
  weather.value = res
  titleCityName.value = res.city
})
</script>

<style lang="scss">
.map-view {
  position: fixed;
  inset: 0;
  .map-header {
    display: flex;
    gap: 2rem;
    padding: 1rem;
    background-color: #4264fb;
    .local-name {
      color: #333;
      text-decoration: none;
      align-content: center;
    }
    form {
      height: 2rem;
      input {
        height: 100%;
      }
    }
  }
}
</style>
