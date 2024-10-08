<script>
  import pin1Icon from '@/assets/pin-1.svg'
  import pin2Icon from '@/assets/pin-2.svg'
  export default {
    data() {
      return {
        zoom: 5,
        tileUrl: "https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png",
        startIcon: new L.Icon({
          iconUrl: pin1Icon,
          iconAnchor: [11, 28],
          popupAnchor: [1, -20],
        }),
        endIcon: new L.Icon({
          iconUrl: pin2Icon,
          iconAnchor: [11, 28],
          popupAnchor: [1, -20],
        }),
      }
    },
    props: {
      center: {
        default: [52.52000659999999, 13.404954]
      },
      routeCoords: {

      },
      loadingSpots: {

      },
      unloadingSpots: {

      }
    },
    computed: {
      startPoint() {
        return this.loadingSpots[0].value ? [this.loadingSpots[0].value.lat, this.loadingSpots[0].value.lng]: []
      },
      endPoint() {
        const lastIndex = this.unloadingSpots.length - 1 
        return this.unloadingSpots[lastIndex].value ? [this.unloadingSpots[lastIndex].value.lat, this.unloadingSpots[lastIndex].value.lng]: []
      },
      additionalSpots() {
         const additionalLoadSpots = this.loadingSpots.length > 1 
          ? this.loadingSpots.slice(1).filter(i => i.value).map(item => [item.value.lat, item.value.lng])
          : []
        
        const additionalUnloadSpots = this.unloadingSpots.length > 1
          ? this.unloadingSpots.slice(0, -1).filter(i => i.value).map(item => [item.value.lat, item.value.lng])
          : []

        return additionalLoadSpots.concat(additionalUnloadSpots)
      }
    }
  }
</script>
<template>
  <div id="map" style="height: 88vh">
    <l-map :zoom="zoom" :center="center" style="height: 100%; width: 100%;">
      <l-tile-layer :url="tileUrl" />
      <l-polyline :lat-lngs="routeCoords" />
      
      <template v-if="startPoint.length" >
        <l-marker :lat-lng="startPoint" :icon="startIcon">
          <l-popup>Starting Point</l-popup>
        </l-marker>
      </template>

      <template v-if="additionalSpots.length">
        <l-marker
          v-for="(s, index) in additionalSpots"
          :key="index"
          :lat-lng="s"
        >
          <l-popup>Additional Spot</l-popup>
        </l-marker>
        
      </template>
      <template v-if="endPoint.length">
        <l-marker :lat-lng="endPoint"  :icon="endIcon">
          <l-popup>End Point</l-popup>
        </l-marker>
      </template>
    </l-map>
  </div>
</template>