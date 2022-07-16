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
      pointToSet: null,
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

        //map.addControl(
        //  new mapboxgl.GeolocateControl({
        //    positionOptions: {
        //      enableHighAccuracy: true,
        //    },
        //    trackUserLocation: true,
        //  })
        //);

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
              "circle-color": "#070808",
            },
            filter: ["in", "$type", "Point"],
          });

          map.on("click", (e) => {
            if (geojson.features.length >= 1) {
              geojson.features.pop();
            }
            // Check if an icon was clicked, if so, do nothing
            //[[e.point - 20,e.point + 20], [e.point + 20, e.point - 20]]
            const features = map.queryRenderedFeatures(e.point);

            if (features.length) {
              return
            } else {
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

              //map.getSource("geojson").setData(geojson);
              const el = document.createElement("div");

              el.style.backgroundImage = `url(/images/placeholder.png)`;
              el.style.width = `40px`;
              el.style.height = `40px`;
              el.style.backgroundSize = "100%";
              el.className = "marker";

              self.pointToSet = point;
              self.$emit("setPoint", point);

              let content = document.createElement("div");
              content.style.width = `$100px`;
              content.style.height = `$100px`;
              //content.appendChild(document.createTextNode("popup"))
              content.appendChild(document.createElement("br"));

              let button = document.createElement("button");
              button.innerHTML = "request to join button";

              content.appendChild(document.createElement("br"));
              content.appendChild(button);
              let popup = new mapboxgl.Popup()
                .setDOMContent(content)
                .setLngLat([e.lngLat.lng, e.lngLat.lat]);

              const marker = new mapboxgl.Marker(el)
                .setLngLat([e.lngLat.lng, e.lngLat.lat])
                .addTo(self.map);

              el.addEventListener("click", () => {
                marker.setPopup(popup);
              });
            }
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
  width: 100%;
}

#mapElement {
  height: 100%;
  width: 100%;
}

.marker {
  display: block;
  border: none;
  border-radius: 50%;
  cursor: pointer;
  padding: 0;
  background-image: url("./rugby.png");
}
</style>
