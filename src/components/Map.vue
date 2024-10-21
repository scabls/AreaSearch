<template>
  <main id="map"></main>
</template>

<script setup>
import { ref, watch, onMounted } from 'vue'
import { getAdcode } from '@/api/weather'
import { Map, View } from 'ol'
import TileLayer from 'ol/layer/Tile'
import { XYZ } from 'ol/source'
import VectorLayer from 'ol/layer/Vector'
import VectorSource from 'ol/source/Vector'
import Style from 'ol/style/Style'
import Fill from 'ol/style/Fill'
import Stroke from 'ol/style/Stroke'
import GeoJSON from 'ol/format/GeoJSON'

const { geodata } = defineProps(['geodata'])

const map = new Map({})
const view = new View({
  projection: 'EPSG:4326',
})
const baseLayer = new TileLayer({
  source: new XYZ({
    url: 'http://wprd0{1-4}.is.autonavi.com/appmaptile?lang=zh_cn&size=1&style=7&x={x}&y={y}&z={z}',
    wrapX: false,
  }),
})
const style = new Style({
  fill: new Fill({
    color: 'rgba(0,0,0,0.2)',
  }),
  stroke: new Stroke({
    color: 'red',
  }),
})
const roiLayer = new VectorLayer({})
roiLayer.setStyle(style)
map.setView(view)
map.addLayer(baseLayer)
map.addLayer(roiLayer)
const renderMap = geodata => {
  //   view.setCenter(geodata.location.split(',').map(Number))
  view.animate({
    center: geodata.location.split(',').map(Number),
  })
  switch (geodata.level) {
    case '国家':
      //   view.setZoom(5)
      view.animate({
        zoom: 5,
      })
      break
    case '省':
      //   view.setZoom(7)
      view.animate({
        zoom: 7,
      })
      break
    case '市':
      //   view.setZoom(9)
      view.animate({
        zoom: 9,
      })
      break
  }
  const source = new VectorSource({
    url() {
      return `https://geo.datav.aliyun.com/areas_v3/bound/${geodata.adcode}.json`
    },
    format: new GeoJSON(),
  })
  roiLayer.setSource(source)
}
watch(
  () => geodata,
  () => {
    renderMap(geodata)
  }
)
onMounted(() => {
  // 注意vue设置状态时模板未加载
  map.setTarget('map')
})
</script>

<style lang="scss" scoped>
#map {
  position: relative;
  height: calc(100% - 4rem);
}
</style>
