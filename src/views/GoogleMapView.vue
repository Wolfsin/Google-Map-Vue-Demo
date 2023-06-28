<script setup lang="ts">
import { Loader } from '@googlemaps/js-api-loader'
import { gcj_wgs } from 'prcoords'
import { ref } from 'vue'
interface MarkerWithLabel extends google.maps.Marker {
  label?: string
}
const markerList: MarkerWithLabel[] = []
const mapRef = ref<HTMLElement>()
const inputValue = ref('')
const loader = new Loader({
  apiKey: import.meta.env.VITE_GOOGLE_API_KEY,
  version: 'weekly',
  libraries: ['places'],
  region: 'JP',
  language: 'zh-CN',
})
const mapOptions = {
  center: {
    lat: 35.6812784717969,
    lng: 139.7668380855873
  },
  zoom: 15,
  mapTypeId: 'hybrid',
}
let map: google.maps.Map
let isChinaFlag: boolean
loader
  .importLibrary('maps')
  .then(({ Map }) => {
    map = new Map(mapRef.value!, mapOptions)
  })
  .catch((e) => {
    console.log(e)
  })
const google = window.google

// action
const handleBlur = () => {
  console.log('inputValue', inputValue.value)
}
const Geocoding = () => {
  const request = {
    query: inputValue.value,
    fields: ['name', 'geometry', 'place_id', 'formatted_address']
  }
  const service = new google.maps.places.PlacesService(map)
  service.findPlaceFromQuery(request, (results, status) => {
    console.log('results', results)
    console.log('status', status)
    if (status === google.maps.places.PlacesServiceStatus.OK && results) {
      isChinaFlag = results[0].formatted_address!.includes('中国')
      console.log('isChina', isChinaFlag)
      createMarker(results[0])
      map.setCenter(results[0].geometry!.location!)
    }
  })
}
const save = () => {
  markerList.forEach((element) => {
    console.log(element.label)
    console.log(element.getPosition()!.toJSON())
  })
}
const createMarker = (place: google.maps.places.PlaceResult) => {
  if (!place.geometry || !place.geometry.location) return
  const fixPosition = gcj_wgs({
    lon: place.geometry.location.lng(),
    lat: place.geometry.location.lat()
  })
  const marker = new google.maps.Marker({
    map,
    position: isChinaFlag
      ? { lat: fixPosition.lat, lng: fixPosition.lon }
      : place.geometry.location,
    draggable: true,
    label: (markerList.length + 1).toString()
  })
  markerList.push(marker)
  marker.addListener('dragend', (dragendEvent: google.maps.MapMouseEvent) => {
    console.log(dragendEvent.latLng!.lat())
    console.log(dragendEvent.latLng!.lng())
  })
}
</script>
<template>
  <div class="about">
    <h3>Simple Map Demo By Google</h3>
    <input v-model="inputValue" type="text" @blur="handleBlur" />
    <button @click="Geocoding">Geocoding</button>
    <button @click="save">save</button>
  </div>
  <div id="map" class="map" ref="mapRef"></div>
</template>

<style>
.map {
  height: 400px;
  width: 100%;
}
</style>
