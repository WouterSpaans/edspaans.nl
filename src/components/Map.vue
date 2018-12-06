<template>
  <div id='map'></div>
</template>

<script>
import 'isomorphic-fetch'
import GoogleMapsLoader from 'google-maps'
import axios from 'axios'
export default {
  name: 'Map',
  data () {
    return {
      OwntracksBase: 'https://owntracks.edspaans.nl/owntracks/api/0/locations',
      OwntracksUser: 'wouter',
      OwntracksDevice: 'samsung-ed',
      OwntracksFielde: 'tst,lat,lon,acc',
      OwntracksFrom: '2018-03-21T09:06:00Z',
      OwntracksTo: '2018-04-25T22:06:00Z',
      OwntracksCoordinates: [],
      google: null
    }
  },
  computed: {
    OwntracksUrl: function () {
      return this.OwntracksBase +
        '?user=' + this.OwntracksUser +
        '&device=' + this.OwntracksDevice +
        '&fields=' + this.OwntracksFielde +
        '&from=' + this.OwntracksFrom +
        '&to=' + this.OwntracksTo +
        '&nocache=' + new Date().getTime()
    }
  },
  watch: {
    OwntracksTo: function () {
      this.UpdateMap()
    },
    OwntracksCoordinates: function (val) {
      let filteredCoords = val.data.filter(function (coord) { return coord.acc <= 100 })
      let map = new this.google.maps.Map(document.getElementById('map'))

      // Set viewport (bounds)
      let bounds = new this.google.maps.LatLngBounds()
      // eslint-disable-next-line
      filteredCoords.forEach(coord => { bounds.extend(new google.maps.LatLng(coord.lat, coord.lon)) })
      map.fitBounds(bounds)

      // Draw the line
      // eslint-disable-next-line
      let polyline = new google.maps.Polyline({
        path: filteredCoords.map(function (e) { return { lat: e.lat, lng: e.lon } }),
        strokeColor: '#FF0000',
        strokeOpacity: 0.6,
        strokeWeight: 5
      })
      polyline.setMap(map)

      // Get all marker indexes
      var markerIndexes = []
      let previousDay = 0
      let currentDay = 0
      for (let index = 1; index < filteredCoords.length; index++) {
        currentDay = new Date(filteredCoords[index].tst * 1000).getDate()
        if (currentDay !== previousDay) {
          previousDay = currentDay
          markerIndexes.push(index - 1)
        }
      }
      markerIndexes.push(filteredCoords.length - 1)

      // Draw all markers
      let daynumber = 0
      markerIndexes.forEach(index => {
        if (daynumber !== 11) {
          const coord = filteredCoords[index]
          // eslint-disable-next-line
          let marker = new google.maps.Marker({
            // eslint-disable-next-line
            position: new google.maps.LatLng(coord.lat, coord.lon),
            label: daynumber.toString(),
            map: map
          })
        }
        daynumber++
      })
    }
  },
  methods: {
    UpdateMap: function () {
      axios.get(this.OwntracksUrl)
        .then(response => {
          this.OwntracksCoordinates = response.data
        })
    },
    OldWay: function () {
      let url = this.OwntracksUrl
      GoogleMapsLoader.KEY = 'AIzaSyC-QX42CVqFcXoxHh-DTJXw6h-6BTuUO2s'
      GoogleMapsLoader.load(function (google) {
        fetch(url)
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
  },
  mounted: function () {
    GoogleMapsLoader.KEY = 'AIzaSyC-QX42CVqFcXoxHh-DTJXw6h-6BTuUO2s'
    var parent = this
    GoogleMapsLoader.load(function (google) {
      parent.google = google
      parent.UpdateMap()
    })
  }
}
</script>

<style scoped>
#map {
  height: 100vh;
}
</style>
