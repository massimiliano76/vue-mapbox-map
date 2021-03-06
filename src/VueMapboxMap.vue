<template lang='pug'>
    div(ref='mapboxMapDiv')
</template>

<script>
  import mapboxgl from 'mapbox-gl'
  import MapboxGeocoder from '@mapbox/mapbox-gl-geocoder'

  export default {
    name: 'VueMapboxMap',
    data () {
      return {
        map: null
      }
    },
    props: {
      // mapbox requires an access token
      'access-token': {
        type: String,
        required: true
      },
      // target map style, you can also load a local map style configuration
      'map-style': {
        type: [String, Object],
        default: 'mapbox://styles/mapbox/light-v9'
      },
      // whether map can be interacted with
      interactive: {
        type: Boolean,
        default: true
      },
      // whether to instance the geocoder
      geocoder: {
        type: Boolean,
        default: false
      },
      // longitude (dynamic)
      lng: {
        type: [Number, String],
        required: true
      },
      // latitude (dynamic)
      lat: {
        type: [Number, String],
        required: true
      },
      // zoom (dynamic)
      zoom: {
        type: [Number, String],
        default: 13
      },
      // pitch (dynamic)
      pitch: {
        type: [Number, String],
        default: 60
      },
      // bearing (dynamic)
      bearing: {
        type: [Number, String],
        default: 0
      }
    },
    mounted () {
      mapboxgl.accessToken = this.accessToken
      this.map = new mapboxgl.Map({
        container: this.$refs.mapboxMapDiv,
        style: this.mapStyle,
        interactive: this.interactive,
        center: [
          this.lng,
          this.lat
        ],
        zoom: this.zoom,
        bearing: this.bearing,
        pitch: this.pitch
      })
      if (this.geocoder) {
        this.map.addControl(new MapboxGeocoder({
          accessToken: this.accessToken,
          zoom: 18,
          flyTo: true,
          // bias results closer to london
          proximity: {
            longitude: this.lng,
            latitude: this.lat
          }
        }))
      }
      // return the map for reference from parent component
      this.map.on('load', () => { this.$emit('mapbox-ready', this.map) })
      this.map.on('dragend', () => { this.$emit('dragend') })
      this.map.on('zoomend', () => { this.$emit('zoomend') })
      this.map.on('remove', () => { this.$emit('mapbox-destroyed') })
    },
    watch: {
      mapStyle: {
        deep: true,
        handler (val) {
          if (val && this.map) {
            // TODO: currently custom data layers are lost, but probably option to save them in future per
            // https://github.com/mapbox/mapbox-gl-js/issues/4006
            // currently this means that updating the style will undo any custom feature collections...
            this.map.setStyle(val)
          } else {
            console.error(`NOTE -> Unable to update map style to ${val}.`)
          }
        }
      },
      lng (val) {
        if (val && this.map && this.lat) {
          this.map.jumpTo({ center: [this.lng, this.lat ] })
        } else {
          console.error(`NOTE -> Unable to update centre to ${this.lng}, ${this.lat}.`)
        }
      },
      lat (val) {
        if (val && this.map && this.lng) {
          this.map.jumpTo({center: [this.lng, this.lat]})
        } else {
          console.error(`NOTE -> Unable to update centre to ${this.lng}, ${this.lat}.`)
        }
      },
      zoom (val) {
        if (val && this.map) {
          this.map.setZoom(val)
        } else {
          console.error(`NOTE -> Unable to update zoom to ${val}.`)
        }
      },
      pitch (val) {
        if (val >= 0 && val <= 60 && this.map) {
          this.map.setPitch(val)
        } else {
          console.error(`NOTE -> Unable to update pitch to ${val}. Exceeds permitted range.`)
        }
      },
      bearing (val) {
        if (val >= 0 && val <= 360 && this.map) {
          this.map.setBearing(val)
        } else {
          console.error(`NOTE -> Unable to update bearing to ${val}. Exceeds permitted range.`)
        }
      }
    }
  }
</script>