<template>

  <div style="height: 100vh; width: 100%" id="app" v-cloak>
    <l-map
      v-if="showMap"
      :zoom="zoom"
      :center="center"
      :options="mapOptions"
      style="height: 100%"
    >
      <l-tile-layer
        :url="url"
        :attribution="attribution"
      />
      <l-control-scale position="topright" :imperial="false" :metric="true"></l-control-scale>
      <l-circle
        :lat-lng="center"
        :radius="circle.radius"
        :stroke="circle.stroke"
        :color="circle.color"
        :weight="circle.weight"
        :fillColor="circle.fillColor"
        />
      <l-circle-marker
        :lat-lng="center"
        :radius="circlemarker.radius"
        :stroke="circle.stroke"
        :fillColor="circlemarker.fillColor"
        :fillOpacity="circlemarker.fillOpacity"
        >
        <l-tooltip :options="{ permanent: true, direction:'top' }">
          🏠
        </l-tooltip>
      </l-circle-marker>
      <l-circle-marker
        :lat-lng="updatedCenter"
        :radius="circlemarker.radius"
        :stroke="circle.stroke"
        :fillColor="circlemarker.fillColor"
        :fillOpacity="circlemarker.fillOpacity"
        >
        <l-tooltip :options="{ permanent: true, direction:'top' }">
          🏃‍♀️
        </l-tooltip>
      </l-circle-marker>
    </l-map>
  </div>
</template>

<script>
import { latLng } from "leaflet";
import { LMap, LTileLayer, LCircleMarker, LCircle, LControlScale, LTooltip } from "vue2-leaflet";

export default {
  name: "Example",
  components: {
    LMap,
    LTileLayer,
    LCircleMarker,
    LCircle,
    LControlScale,
    LTooltip
  },
  data() {
    return {
      markers: [],
      places: [],
      currentPlace: null,
      zoom: 15,
      center: latLng(40.3915307,-3.6974064),
      url: 'https://{s}.basemaps.cartocdn.com/rastertiles/voyager/{z}/{x}/{y}{r}.png',
      attribution:
        '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors &copy; <a href="https://carto.com/attributions">CARTO</a>',
      showParagraph: false,
      updatedCenter: [40.3928565,-3.6992007],
      circle: {
        center: [40.3915307,-3.6974064],
        radius: 1000,
        color: '#1CD89A',
        weight: 2,
        fillColor: '#1CD89A'
      },
      circlemarker: {
        center: [40.3915307,-3.6974064],
        radius: 6,
        stroke: false,
        fillColor: '#006DDC',
        fillOpacity: 1
      },
      mapOptions: {
        zoomSnap: 0.5
      },
      showMap: true
    };
  },
  mounted() {
    this.geolocate();
    this.watchposition();
  },
  methods:  {
    geolocate:  function()  {
      navigator.geolocation.getCurrentPosition(position => {
        this.center = {
          lat: position.coords.latitude,
          lng: position.coords.longitude
        };
      });
    },
    watchposition:  function()  {
      navigator.geolocation.watchPosition(position => {
        this.updatedCenter = {
          lat: position.coords.latitude,
          lng: position.coords.longitude
        };
      });
    }
  }
};
</script>

<style>
body {
  margin: 0;
  padding: 0;
  height: 100%;
  width: 100%;
}
#app {
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
.leaflet-tooltip {
  font-size: 36px;
  padding: 4px 10px !important
}
</style>
