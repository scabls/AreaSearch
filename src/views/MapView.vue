<template>
  <div class="map-view">
    <header class="map-header">
      <router-link to="/" class="local-name">{{ titleAddress }}</router-link>
      <form @submit.prevent="handleSearch" class="search-form">
        <input type="text" placeholder="搜索城市" v-model="keyword" class="search-input" />
      </form>
      <button @click="resetMap" class="reset-btn">复位地图</button>
      <el-select v-model="drawType" placeholder="选择绘制类型" style="width: 240px">
        <el-option
          v-for="item in drawTypes"
          :key="item.value"
          :label="item.label"
          :value="item.value"
        />
      </el-select>
      <el-select
        v-model="iconType"
        placeholder="选择标记类型"
        style="width: 240px"
        :disabled="!canAddMarker"
      >
        <el-option
          v-for="item in iconTypes"
          :key="item.value"
          :label="item.label"
          :value="item.value"
        />
      </el-select>
    </header>
    <Map :geodata :drawType :iconType v-model="canAddMarker" />
    <WeatherInfo :weather />
  </div>
</template>

<script setup>
import Map from '@/components/Map.vue'
import WeatherInfo from '@/components/WeatherInfo.vue'

import { ref, onMounted } from 'vue'
import { useRoute } from 'vue-router'
import { getWeather, getAdcode } from '@/api/weather'
import { disable } from 'ol/rotationconstraint'

const route = useRoute()
const { adcode } = route.params

const titleAddress = ref('')
const weather = ref({})
const geodata = ref({})
const keyword = ref('')
const drawType = ref('')
const iconType = ref('')
const canAddMarker = ref(false)
const drawTypes = [
  { value: 'LineString', label: '折线' },
  { value: 'Circle', label: '圆' },
  { value: 'Polygon', label: '多边形' },
  { value: 'stopDraw', label: '退出绘制' },
  { value: 'clearDraw', label: '清除绘制' },
]
const iconTypes = ref([
  { value: 'bus', label: '公交车站' },
  { value: 'charge', label: '充电站' },
  { value: 'park', label: '停车场' },
  { value: 'stopMark', label: '停止标记' },
  { value: 'clearMark', label: '清除标记' },
])

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
        color: #a8abb2;
      }
    }
    .reset-btn {
      background-color: #fff;
      padding-left: 1rem;
      padding-right: 1rem;
      border-radius: 0.3rem;
      color: #a8abb2;
      cursor: pointer;
    }
    .el-select {
      height: 2.5rem;
      .el-select__wrapper {
        height: 100%;
      }
    }
  }
}
</style>
