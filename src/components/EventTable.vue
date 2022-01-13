<template>
  <div>
    {{ area_geojson }}
    <table>
      <tr v-for="feature in area_geojson.features" :key="feature.id">
        <td>{{ feature.properties.message }}</td>
      </tr>
    </table>
  </div>
</template>

<script>
export default {
  props: ["area_geojson"],
}
</script>

<style>
table{
  width: 100%;
  border-collapse: collapse;
  border-spacing: 0;
}

table th,table td{
  padding: 10px 0;
  text-align: center;
}

table tr:nth-child(odd){
  background-color: #eee
}
</style>