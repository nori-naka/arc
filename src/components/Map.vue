<template>
  <div>
    <div class="map_layout">
      <div  class="map" id="map"></div>
      <div v-show="polygon_pt.length > 0" @click="remove_point" class="btn_menu">
        <img :src="img_back" class="img_back" />
      </div>
    </div>
    <modal v-show="show_modal_flag" @hide_modal="hide_modal" :polygon_props="polygon_props" />
  </div>
</template>

<script>
import "leaflet/dist/leaflet.css"
import L from "leaflet";
import Modal from "./Modal.vue";
import { uuidv4 } from "./uuidv4";

// const FEATURECOLLECTION = { "type": "FeatureCollection", "features": []};
const GEO_URL = "https://lma1.herokuapp.com/geojson";
// const GEO_URL = "http://localhost:3000/geojson";

export default {
  components: {
    Modal
  },
  data() {
    return {
      map: null,
      coords: { lat: "", lng: "", time_stamp: 0 },
      last_coords: { lat: "", lng: "", time_stamp: 0 },
      self_marker: null,
      polygon_pt: [],
      polygon_coords: [],
      polygon_marker: [],
      polygons: {},
      // area_events: null,
      pt_mode: true,
      featureCollection_json: {},
      features_obj: {},
      show_modal_flag: false,
      polygon_props: {},
      img_back: require("../assets/arrow_left.png")
    }
  },
  async mounted() {
    // this.area_events = L.featureGroup();
    this.init_map();
    this.map.on("keydown", ev => {
      if (ev.originalEvent.key == "Escape") {
        ev.originalEvent.preventDefault();
        console.log("Escape");
        this.remove_point();
      }
    });
  },
  computed: {
  },
  methods: {
    hide_modal(ev) {
      this.show_modal_flag = false;
      console.log(ev);
      // this.features_obj[]
      if (ev.id == null) return;

      this.map.eachLayer(layer => {
        if (layer.id == ev.id) {
          console.log("あったよ");
          layer.bindPopup(`<div>${ev.value}</div>`);
          layer.openPopup();

          this.features_obj[ev.id].properties.message = ev.value;
          this.features_obj[ev.id].properties.msg_id = ev.msg_id;
          this.update_featureCollection();

          this.polygon_props = {};
          return
        }
      });
    },
    async update_featureCollection(data) {
      if (data && data.type == "FeatureCollection") {
        this.featureCollection_json = data;
      } else {
        // 各featureを合体
        // FEATURECOLLECTION.features = Object.values(this.features_obj);
        this.featureCollection_json = {
          "type": "FeatureCollection", 
          "features": [ ...Object.values(this.features_obj) ]
        }
      }
      console.log(this.featureCollection_json);
      this.$emit("update_area", this.featureCollection_json);

      const res = await fetch(GEO_URL, {
        method: "POST",
        headers: {
          "Accept": "application/json",
          "Content-Type": "application/json"
        },
        body: JSON.stringify(this.featureCollection_json)
      });
      if (res.ok) {
        console.log("POST OK");
      }
    },
    async init_map() {
      navigator.geolocation.watchPosition(this.geo_success, this.geo_error);

      const osmLayer = L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',{
        attribution: '© <a href="http://osm.org/copyright">OpenStreetMap</a> contributors, <a href="http://creativecommons.org/licenses/by-sa/2.0/">CC-BY-SA</a>',
        opacity: 0.5
      });
      // 地図読み込み
      this.map = L.map("map", {
        center: L.latLng( 35.6825, 139.752778), 
        zoom: 15
      }).addLayer(osmLayer);

      // サーバの現状のgeoJSONの読み込み
      const res = await fetch(GEO_URL);
      const geo_data = await res.json();
      this.update_featureCollection(geo_data);
      L.geoJSON(geo_data, {
        onEachFeature: (feature, layer) => {
          console.log(layer);
          const id = feature.properties.id;
          layer.id = id;
          this.polygons[id] = layer;
          this.features_obj[id] = feature;
        }
      }).bindPopup( layer => {
        return layer.feature.properties.message;
      }).addTo(this.map);

      this.map.on("click", ev => {
        // 最後にpt追加された場所が最初のpt追加した場所に近いか（画面PXで10以内）を判断してpolygonを閉じる
        if (this.polygon_pt.length > 0 &&  ev.layerPoint.distanceTo(this.polygon_pt[0]) < 30) {
          const _polygon = L.polygon(this.polygon_coords).addTo(this.map).on("preclick", ev => {
            console.log("---------EV----------");
            console.log(ev);
            console.log(ev.sourceTarget.id);
            // 最初のPT追加で他のpolygonの内部の場合には、PT追加はしないで、eventの追加を行う
            if (this.polygon_pt.length == 0) {
              this.pt_mode = false;
              this.show_modal_flag = true;
              this.polygon_props = {
                id: ev.sourceTarget.id,
                // latlng: ev.latlng
              }
            }
          });
          // polygonを閉じたので、markerは削除する
          this.polygon_marker.forEach(marker => {
            this.map.removeLayer(marker);
          });


          const feature_geojson = _polygon.toGeoJSON();
          const id = uuidv4();
          const msg_id = uuidv4();
          _polygon.id = id;
          this.polygons[id] = _polygon;
          feature_geojson.properties["id"] = id;
          feature_geojson.properties["msg_id"] = msg_id;
          this.features_obj[id] = feature_geojson;

          // 最後なので、後片付け
          this.polygon_marker = [];
          this.polygon_coords = [];
          this.polygon_pt = [];
        } else {
          if (this.pt_mode) {
            this.add_point(ev);
          } else {
            this.pt_mode = true;
          }
        }
        console.log(this.polygon_pt);
      });
    },
    async geo_success(pos) {
      this.coords = {
        lat: Math.round(pos.coords.latitude * 1000000) / 1000000,
        lng: Math.round(pos.coords.longitude * 1000000) / 1000000,
        time_stamp: Date.now()
      }
      const latlng = L.latLng(this.coords);
      if (this.first_flag) {
        this.map.panTo(latlng, {animate: true});
        this.first_flag = false;
      }

      if (this.self_marker) {
        this.self_marker.setLatLng(latlng);
      } else {
        // アイコンを使用する場合
        this.self_marker = L.marker(latlng, {
          icon: L.icon({
            className: "icon_style",
            iconUrl: require("@/assets/people_marker.png"),
            iconSize: [40, 40],
            iconAnchor: [20, 20],
            popupAnchor: [0, -20]
          })
        })
          .addTo(this.map)
     }
    },
    geo_error(error) {
      console.log(`GEO_ERROR: ${error.message}`);
    },
    add_point(ev) {
      console.log(ev);
      const marker = L.marker(ev.latlng, {
        icon: L.divIcon({
          html: "<div></div>",
          className: "point_marker"
        })
      }).addTo(this.map);

      this.polygon_marker.push(marker);
      this.polygon_coords.push(ev.latlng);
      this.polygon_pt.push(ev.layerPoint);
    },
    remove_point() {
      if (this.polygon_pt.length > 0 && this.polygon_coords.length > 0) {
        this.polygon_coords.pop();
        this.polygon_pt.pop();
        const marker = this.polygon_marker.pop();
        this.map.removeLayer(marker);
      }
    },
    edit_polygon(id) {
      this.show_modal_flag = true;
      this.polygon_props = {
        id: id,
        value: this.features_obj[id].properties.message ? this.features_obj[id].properties.message : ""
        // latlng: null
      }
    },
    delete_polygon(id) {
      this.map.removeLayer(this.polygons[id]);
      delete this.polygons[id];
      delete this.features_obj[id];
      this.update_featureCollection();
    }
  }
}
</script>

<style>
.map_layout {
  position: relative;
  top: 0px;
  left: 0px;
  height: 50vh;
  width: 100%;
}
.map {
  position: relative;
  /* top: 0px;
  left: 0px; */
  height: 100%;
  width: 100%;
  z-index: 1;
}
.btn_menu {
  display: flex;
  position: absolute;
  bottom: 10px;
  left: 10px;
  width: 50px;
  height: 50px;
  background-color: darkturquoise;
  border-radius: 50%;
  z-index: 2;
  cursor: pointer;
}
.img_back {
  width: 60%;
  height: 60%;
  margin: auto;
}
.point_marker {
  width: 10px;
  height: 10px;
  border-radius: 50%;
  background-color: blue;
}
</style>