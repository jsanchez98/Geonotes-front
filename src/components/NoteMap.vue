<template>
  <div id="mapContainer">
    <div id="mapElement"></div>
  </div>
</template>

<script>
import mapboxgl from "mapbox-gl";
import { nextTick } from "vue";

export default {
  name: "NoteMap",
  data() {
    return {
      map: null,
      createLocation: null,
    };
  },
  created() {
    this.renderMap();
  },
  methods: {
    renderMap() {
      let self = this;
      nextTick(function () {
        mapboxgl.accessToken =
          "pk.eyJ1IjoianNhbmNoOTgiLCJhIjoiY2w1Z3Q1Yzc4MDJiODNrdGVqd3Rtb2V5OSJ9.GYwWTRDCQn85XZVUEU76EA";
        let map = new mapboxgl.Map({
          container: "mapElement",
          projection: "globe",
          style: "mapbox://styles/mapbox/streets-v11",
          center: [-2.47657, 51.279816],
          zoom: 13,
        });
        self.map = map;

        const geojson = {
          type: "FeatureCollection",
          features: [],
        };

        map.addControl(
          new mapboxgl.GeolocateControl({
            positionOptions: {
              enableHighAccuracy: true,
            },
            trackUserLocation: true,
          })
        );

       //map.addLayer({
       //  id: "point",
       //  type: "circle",
       //  source: "geojson",
       //  paint: {
       //    "circle-radius": 7,
       //    "circle-color": "#2638ff",
       //  },
       //  filter: ["in", "$type", "Point"],
       //});

        map.on("load", () => {
          map = self.map;
          map.addSource("geojson", {
            type: "geojson",
            data: geojson,
          });

          map.addLayer({
            id: "point",
            type: "circle",
            source: "geojson",
            paint: {
              "circle-radius": 7,
              "circle-color": "#2638ff",
            },
            filter: ["in", "$type", "Point"],
          });

          map.on("click", (e) => {
            if (geojson.features.length >= 1) {
              geojson.features.pop();
            }

            const point = {
              type: "Feature",
              geometry: {
                type: "Point",
                coordinates: [e.lngLat.lng, e.lngLat.lat],
              },
              properties: {
                id: String(new Date().getTime()),
              },
            };

            geojson.features.push(point);

            map.getSource("geojson").setData(geojson);

            self.createLocation = point;
          });
        });
      });
    },
  },
};
</script>

<style scoped>
#mapContainer {
  height: 100%;
  width: 1000px;
}

#mapElement {
  height: 100%;
  width: 1000px;
}
</style>
