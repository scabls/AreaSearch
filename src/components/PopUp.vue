<template>
  <div class="popup" ref="popup">
    <p>地区名: {{ geodata.formatted_address }}</p>
    <p>区域等级: {{ geodata.level }}</p>
    <p>经度{{ coordinate[0] }}</p>
    <p>纬度: {{ coordinate[1] }}</p>
  </div>
</template>

<script setup>
import { useTemplateRef, onMounted, computed } from 'vue'

const popUpE = defineModel()
const { geodata } = defineProps({
  geodata: {
    type: Object,
    required: true,
  },
})

const popUpRef = useTemplateRef('popup')

const coordinate = computed(() => {
  if (!geodata.location) return []
  return geodata.location.split(',')
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
