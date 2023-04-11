<template>
<!--  <GaodeMap/>-->
  <Map/>
</template>

<script>
import GaodeMap from '@/components/GaodeMap'
import Map from "@/components/Map";

export default {
  name: 'App',
  components: {
    Map,
    GaodeMap
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
