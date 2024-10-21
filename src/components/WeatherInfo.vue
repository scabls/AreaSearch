<template>
  <aside class="info-aside" v-show="weather.city">
    <table>
      <tr>
        <td>城市</td>
        <td>{{ weather.city }}</td>
      </tr>
      <tr>
        <td>天气</td>
        <td>{{ weather.weather }}</td>
      </tr>
      <tr>
        <td>温度</td>
        <td>{{ weather.temperature }}°C</td>
      </tr>
      <tr>
        <td>风向</td>
        <td>{{ weather.winddirection }}</td>
      </tr>
      <tr>
        <td>风力</td>
        <td>{{ weather.windpower }}</td>
      </tr>
    </table>
  </aside>
</template>

<script setup>
const { weather } = defineProps({
  weather: {
    type: Object,
    required: true,
  },
})
</script>

<style lang="scss" scoped>
.info-aside {
  position: fixed;
  right: 20px;
  top: 70px;
  z-index: 100;
  border-radius: 15px;
  overflow: hidden;
  box-shadow: 0 0 3px 2px #333;
  background: linear-gradient(to top, #6dd5edb0, #2193b0);
  table {
    color: #fff;
    text-align: center;
    width: 200px;
    line-height: 30px;
    border-radius: 15px;
    tr {
      td {
        border: 1px solid rgba(238, 238, 238, 0.473);
      }
    }
  }
}
</style>
