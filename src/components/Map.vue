<template>
  <div id='map'></div>
</template>

<script>
import 'isomorphic-fetch'
import GoogleMapsLoader from 'google-maps'
export default {
  name: 'Map',
  data () {
    return {}
  },
  mounted: function () {
    GoogleMapsLoader.KEY = 'AIzaSyC-QX42CVqFcXoxHh-DTJXw6h-6BTuUO2s'
    GoogleMapsLoader.load(function (google) {
      let urlPart = 'http://owntracks.edspaans.nl/owntracks/api/0/locations?user=wouter&device=samsung-ed&fields=tst,lat,lon,acc'
      let datePart = '&from=2018-03-21T09:06:00Z'
      fetch(urlPart + datePart + '&nocache=' + new Date().getTime())
        .then(res => res.json())
        .then((out) => {
          let map = new google.maps.Map(document.getElementById('map'))
          let bounds = new google.maps.LatLngBounds()
          let filteredCoords = out.data.filter(function (coord) { return coord.acc <= 100 })
          let daynumber = 0
          let previousDay = 0
          let currentDay = 0

          // Draw the line
          let polyline = new google.maps.Polyline({
            path: filteredCoords.map(function (e) { return { lat: e.lat, lng: e.lon } }),
            strokeColor: '#FF0000',
            strokeOpacity: 0.6,
            strokeWeight: 5
          })

          // Add all the markers for the days
          for (let index = 0; index < filteredCoords.length; index++) {
            const coord = filteredCoords[index]
            bounds.extend(new google.maps.LatLng(coord.lat, coord.lon))
            currentDay = new Date(coord.tst * 1000).getDate()
            if (currentDay !== previousDay && index !== 0) {
              previousDay = currentDay
              const previousCoord = filteredCoords[index - 1]
              // eslint-disable-next-line
              let marker = new google.maps.Marker({
                position: new google.maps.LatLng(previousCoord.lat, previousCoord.lon),
                label: daynumber.toString(),
                map: map
              })
              daynumber++
            }
          }

          // Add the last one
          const lastCoord = filteredCoords[filteredCoords.length - 1]
          var image = 'https://developers.google.com/maps/documentation/javascript/examples/full/images/beachflag.png'
          // eslint-disable-next-line
          let marker = new google.maps.Marker({
            position: new google.maps.LatLng(lastCoord.lat, lastCoord.lon),
            map: map,
            icon: image,
            title: 'Huidige positie'
          })

          map.fitBounds(bounds)
          polyline.setMap(map)
        })
    })
  }
}
</script>

<style scoped>
#map {
  height: 100vh;
}
</style>
