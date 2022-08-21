<template>
  <div id="mapContainer">
    <div id="mapElement"></div>
  </div>
</template>

<script>
import mapboxgl from "mapbox-gl";
import { nextTick } from "vue";
import icon from "./placeholder.png";

export default {
  name: "NoteMap",
  data() {
    return {
      map: null,
      pointToSet: null,
    };
  },
  props: {
    posts: [Object],
    focusedNote: null
  },
  created() {
    this.renderMap();
  },
  methods: {
    convertIconsToFeatures(posts) {
      return posts.map((post) => {
        return {
          type: "Feature",
          geometry: {
            type: "Point",
            coordinates: JSON.parse(post.coordinates),
          },
          properties: {
            id: "post.id",
          },
        };
      });
    },
    addIconsToMap(map, reload = false) {
      let geojson = {
        type: "FeatureCollection",
        features: [],
      };
      const icon_features = this.convertIconsToFeatures(this.posts);
      geojson = {
        type: "FeatureCollection",
        features: icon_features,
      };
      
      map.loadImage(icon, (error, image) => {
        if (error) throw error;
        map.addImage("icon", image);
      });
      
      if (reload === false) {
        map.addSource("geojson", {
          type: "geojson",
          data: geojson,
        });
        
        map.addLayer({
          id: "point",
          type: "symbol",
          source: "geojson",
          layout: {
            "icon-image": "icon",
            "icon-size": 0.05,
          },
          filter: ["in", "$type", "Point"],
        });
      } else if (reload === true) {
        map.getSource("geojson").setData(geojson);
      }
      
      return geojson;
    },
    mapTo5dp(list) {
      return list.map((number) => {
        if (typeof number === "string") {
          number = parseFloat(number);
        }
        return number.toFixed(4);
      });
    },
    openNote(features) {
      let component = this;
      if (features.length == 1) {
        alert("5")
        const feature = features[0];
        const note = component.posts.find((post) => {
          let length = post.coordinates.length;
          let candidatePost = component
            .mapTo5dp(post.coordinates.slice(1, length - 1).split(","))
            .toString();
          let searchPost = component
            .mapTo5dp(feature.geometry.coordinates)
            .toString();
          return candidatePost == searchPost;
        });
        alert(JSON.stringify(note))
        this.$emit("setNote", note);
      }
    },
    createNote(popup) {
      let component = this;
      let Popup = popup;
      return function () {
        component.$emit("createNote", component.pointToSet, Popup);
      };
    },
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

        let geolocate = null;

        map.addControl(
          (geolocate = new mapboxgl.GeolocateControl({
            positionOptions: {
              enableHighAccuracy: true,
            },
            trackUserLocation: false,
          }))
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
          const geojson = self.addIconsToMap(map);
          geolocate.trigger();
          let features = [];

          map.on("click", (e) => {
            //if (geojson.features.length >= 1) {
            //  geojson.features.pop();
            //}
            // Check if an icon was clicked, if so, do nothing
            //[[e.point - 20,e.point + 20], [e.point + 20, e.point - 20]]
            features = [];
            alert("1")
            self.$emit("endCreateNote");
            alert("2")
            features = map.queryRenderedFeatures(e.point);
            alert(JSON.stringify(features))
            if (features.length) {
              alert("4")
              self.openNote(features);
              return;
            } else {
              alert("3")
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

              let content = document.createElement("div");
              content.style.width = `$100px`;
              content.style.height = `$100px`;
              //content.appendChild(document.createTextNode("popup"))

              content.appendChild(document.createElement("br"));

              let button = document.createElement("button");
              button.innerHTML = "Create note";

              content.appendChild(document.createElement("br"));
              content.appendChild(button);
              const popup = new mapboxgl.Popup()
                .setDOMContent(content)
                .setLngLat([e.lngLat.lng, e.lngLat.lat])
                .addTo(map);

              popup.on("close", () => {
                self.$emit("endCreateNote");
                map.getSource("geojson").setData(geojson);
              });

              button.addEventListener("click", self.createNote(popup));

              geojson.features.push(point);

              map.getSource("geojson").setData(geojson);

              self.pointToSet = point;
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
}
</style>
