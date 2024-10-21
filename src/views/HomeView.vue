<template>
  <article>
    <dl>
      <dt>城市</dt>
      <dd @click="showMap">北京</dd>
    </dl>
  </article>
</template>

<script setup>
import { useRouter } from 'vue-router'

const router = useRouter()

const showMap = () => {
  router.push({ name: 'map', params: { adcode: 110000 } })
}
</script>

<style lang="scss" scoped>
article {
  padding: 1rem;
  dl {
    margin: 0 auto;
    width: 5rem;
    border: 1px solid black;
    text-align: center;
    dt,
    dd {
      padding: 0.5rem 1rem;
      background-color: beige;
      border-bottom: 1px solid black;
    }
    dd {
      cursor: pointer;
      &:last-child {
        border-bottom: none;
      }
    }
  }
}
</style>
