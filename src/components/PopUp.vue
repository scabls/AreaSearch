<template>
  <div class="popup" ref="popup">
    <p>地区名: {{ popUpData.name }}</p>
    <p>区域等级: {{ areaLevel }}</p>
    <p>经度: {{ coordinate[0] }}</p>
    <p>纬度: {{ coordinate[1] }}</p>
  </div>
</template>

<script setup>
import { useTemplateRef, onMounted, computed } from 'vue'

const popUpE = defineModel()
const { popUpData } = defineProps({
  popUpData: {
    type: Object,
    required: true,
  },
})
const popUpRef = useTemplateRef('popup')

const coordinate = computed(() => {
  if (!popUpData.center) return []
  return popUpData.center.map(n => n.toFixed(2))
})
const areaLevel = computed(() => {
  if (!popUpData.level) return ''
  switch (popUpData.level) {
    case 'country':
      return '全国'
    case 'province':
      return '省级'
    case 'city':
      return '市级'
  }
})

onMounted(() => {
  popUpE.value = popUpRef.value
})
</script>

<style lang="scss" scoped>
.popup {
  display: grid;
  gap: 0.4rem;
  background-color: rgb(238, 238, 238, 0.91);
  padding: 0.5rem 1rem;
  font-size: 0.9rem;
  color: #333;
  border-radius: 0.5rem;
  box-shadow: 0px 0px 10px rgba(0, 0, 0, 0.1);
}
</style>
