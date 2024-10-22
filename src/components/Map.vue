<template>
  <main id="map" :style="{ cursor }"></main>
  <PopUp v-model="popUpE" :geodata />
</template>

<script setup>
import PopUp from './PopUp.vue'
import { ref, watch, onMounted, useTemplateRef } from 'vue'
import { getAdcode } from '@/api/weather'
import { Map, Overlay, View } from 'ol'
import TileLayer from 'ol/layer/Tile'
import { XYZ } from 'ol/source'
import VectorLayer from 'ol/layer/Vector'
import VectorSource from 'ol/source/Vector'
import Style from 'ol/style/Style'
import Fill from 'ol/style/Fill'
import Stroke from 'ol/style/Stroke'
import GeoJSON from 'ol/format/GeoJSON'
import Select from 'ol/interaction/Select.js'
import { pointerMove } from 'ol/events/condition'

const { geodata } = defineProps({
  geodata: {
    type: Object,
    required: true,
  },
})

const popUpE = ref(null)
const cursor = ref('default')

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
const roiLayer = new VectorLayer({
  style,
})
const popUp = new Overlay({})
const select = new Select({
  condition: pointerMove,
})

map.setView(view)
map.addLayer(baseLayer)
map.addLayer(roiLayer)
map.addOverlay(popUp)
map.addInteraction(select)

select.on('select', function (e) {
  const feature = e.selected[0]
  if (feature) {
    popUp.setPosition(feature.getProperties().center)
    cursor.value = 'pointer'
    return
  } else {
    popUp.setPosition(undefined)
    cursor.value = 'default'
  }
})

const renderMap = geodata => {
  const center = geodata.location.split(',').map(Number)
  view.animate({
    center,
  })
  switch (geodata.level) {
    case '国家':
      view.animate({
        zoom: 4,
      })
      break
    case '省':
      view.animate({
        zoom: 7,
      })
      break
    case '市':
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
  // 子组件挂载后获取dom的ref
  popUp.setElement(popUpE.value)
})
</script>

<style lang="scss" scoped>
#map {
  position: relative;
  height: calc(100% - 3rem);
}
</style>
