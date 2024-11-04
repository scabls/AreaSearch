<template>
  <main id="map" :style="{ cursor }"></main>
  <PopUp v-model="popUpE" :popUpData />
</template>

<script setup>
import PopUp from './PopUp.vue'
import { ref, watch, onMounted, useTemplateRef } from 'vue'
import { getAdcode } from '@/api/weather'
import { Feature, Map, Overlay, View } from 'ol'
import TileLayer from 'ol/layer/Tile'
import { Cluster, XYZ } from 'ol/source'
import VectorLayer from 'ol/layer/Vector'
import VectorSource from 'ol/source/Vector'
import Style from 'ol/style/Style'
import Fill from 'ol/style/Fill'
import Stroke from 'ol/style/Stroke'
import GeoJSON from 'ol/format/GeoJSON'
import Select from 'ol/interaction/Select.js'
import Draw from 'ol/interaction/Draw.js'
import { pointerMove } from 'ol/events/condition'
import busIcon from '@/assets/bus.png'
import chargeIcon from '@/assets/charge.png'
import parkIcon from '@/assets/park.png'
import { Point } from 'ol/geom'
import Icon from 'ol/style/Icon'
import CircleStyle from 'ol/style/Circle.js'
import Text from 'ol/style/Text'

let drawer
const icons = { busIcon, chargeIcon, parkIcon }

const canAddMarker = defineModel()

const { geodata, drawType, iconType } = defineProps({
  geodata: { type: Object, required: true },
  drawType: { type: String, required: true },
  iconType: { type: String, required: true },
})

const popUpE = ref(null)
const popUpData = ref({})
const cursor = ref('default')
const layerVisible = ref(true)

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
const drawLayer = new VectorLayer({
  source: new VectorSource(),
  style,
})
const setClusterStyle = feature => {
  const markerCount = feature.get('features').length // != markerSource.getFeatures().length
  const style = new Style()
  if (markerCount === 1) {
    // style.setImage(
    //   // new Icon({
    //   //   src: feature.get('features')[0].get('icon'),
    //   // })
    //   feature.get('features')[0].getStyle().getImage()
    // )
    return feature.get('features')[0].getStyle()
  } else {
    style.setImage(
      new CircleStyle({
        radius: 15,
        stroke: new Stroke({
          color: '#fff',
        }),
        fill: new Fill({
          color: '#3399CC',
        }),
      })
    )
    style.setText(
      new Text({
        text: markerCount.toString(),
        fill: new Fill({
          color: '#fff',
        }),
      })
    )
  }
  return style
}
const markerSource = new VectorSource()
const markerClusterSource = new Cluster({
  source: markerSource,
  distance: 40,
})
const markerLayer = new VectorLayer({
  source: markerClusterSource,
  style: f => setClusterStyle(f),
})
const popUp = new Overlay({})
const select = new Select({
  condition: pointerMove,
})

map.setView(view)
map.addLayer(baseLayer)
map.addLayer(roiLayer)
map.addLayer(drawLayer)
map.addLayer(markerLayer)
map.addOverlay(popUp)
map.addInteraction(select)

select.on('select', function (e) {
  const feature = e.selected[0]
  if (feature) {
    popUpData.value = feature.getProperties()
    popUp.setPosition(popUpData.value.center)
    cursor.value = 'pointer'
    return
  } else {
    popUpData.value = {}
    popUp.setPosition(undefined)
    cursor.value = 'default'
  }
})

view.on('change:resolution', () => {
  const zoom = view.getZoom()
  if (zoom >= 12) {
    layerVisible.value = false
    canAddMarker.value = true
  } else {
    layerVisible.value = true
    canAddMarker.value = false
  }
})

map.on('click', e => {
  if (!canAddMarker.value) return
  if (!iconType) return
  if (iconType === 'stopMark') return
  if (iconType === 'clearMark') return
  const coordinate = e.coordinate
  const marker = new Feature({
    geometry: new Point(coordinate),
  })
  // marker.set('icon', icons[iconType + 'Icon'])
  // marker.setProperties({ icon: icons[iconType + 'Icon'] })
  marker.setStyle(new Style({ image: new Icon({ src: icons[iconType + 'Icon'] }) }))
  markerSource.addFeature(marker)
})

const renderMap = geodata => {
  drawLayer.getSource().clear()
  markerSource.clear()
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
watch(
  () => drawType,
  () => {
    switch (drawType) {
      case 'clearDraw':
        drawLayer.getSource().clear()
      case 'stopDraw':
        if (drawer) {
          drawer.setActive(false)
        }
        break
      default:
        if (drawer) map.removeInteraction(drawer)
        drawer = new Draw({
          type: drawType,
          source: drawLayer.getSource(),
        })
        map.addInteraction(drawer)
    }
  }
)
watch(layerVisible, () => {
  if (layerVisible.value) {
    drawLayer.setVisible(true)
    roiLayer.setVisible(true)
  } else {
    drawLayer.setVisible(false)
    roiLayer.setVisible(false)
  }
})
watch(
  () => iconType,
  () => {
    if (iconType === 'clearMark') markerSource.clear()
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
