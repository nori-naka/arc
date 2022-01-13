<template>
  <div>
    <Map ref="map" @update_area="update_area" />
    <!-- <event-table ref="et" :area_geojson="area_geojson" /> -->
    <table>
      <thead>
        <th>項番</th>
        <th>メッセージ内容</th>
        <th>削除</th>
      </thead>
      <tbody>
        <tr v-for="(feature, index) in features" :key="feature.properties.id">
          <td>{{ index + 1 }}</td>
          <td><a href="#" @click="edit(feature.properties.id)">{{ feature.properties.message }}</a></td>
          <td @click="delete_area(feature.properties.id)"><img :src="img_x"></td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
// import EventTable from './components/EventTable.vue';
import Map from './components/Map.vue'

export default {
  name: 'App',
  components: {
    Map,
    // EventTable
  },
  data() {
    return {
      // area_geojson: null
      features: {},
      img_x: require("./assets/icon_x_red.png")
    }
  },
  methods: {
    update_area(ev) {
      this.features = ev.features;
    },
    edit(id) {
      this.$refs.map.edit_polygon(id);
    },
    delete_area(id) {
      this.$refs.map.delete_polygon(id);
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
body {
  padding: 0px;
  margin: 0px;
}

table{
  width: 100%;
  border-collapse: collapse;
  border-spacing: 0;
}
table th{
  padding: 10px 0;
  text-align: center;
}
table td:nth-child(1),
table td:last-child{
  padding: 10px 0;
  text-align: center;
}
table td{
  padding: 10px 0;
  text-align: left;
}
table tr:nth-child(odd){
  background-color: #eee
}
</style>
