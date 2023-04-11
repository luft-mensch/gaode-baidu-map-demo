<template>
  <baidu-map class="bm-view" :zoom="15" center="北京" :scroll-wheel-zoom="true">
    <bm-navigation anchor="BMAP_ANCHOR_TOP_RIGHT"></bm-navigation>
    <bm-scale anchor="BMAP_ANCHOR_BOTTOM_LEFT"></bm-scale>
    <bm-map-type :map-types="['BMAP_NORMAL_MAP', 'BMAP_HYBRID_MAP']" anchor="BMAP_ANCHOR_TOP_LEFT"></bm-map-type>
    <bm-geolocation anchor="BMAP_ANCHOR_BOTTOM_RIGHT" :showAddressBar="true" :autoLocation="true"></bm-geolocation>
    <bm-polyline :path="polylinePath" stroke-color="blue" :stroke-opacity="0.5" :stroke-weight="2" :editing="true"></bm-polyline>
  </baidu-map>
</template>

<script>
export default {
  name: "Map",
  data(){
     return{
       polylinePath:[
         {lng: 113.94383, lat: 22.555423},
         {lng: 113.949085, lat: 22.554808},
         {lng: 113.947951, lat: 22.552864}
       ],
     }
  },
}
</script>

<style scoped>
.bm-view{
  width: 100%;
  height: 100%;
  position: absolute;
  left: 0;
  top: 0;
}

</style>
