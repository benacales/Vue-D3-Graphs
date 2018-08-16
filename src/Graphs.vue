<template>
  <div id="app" class = "gridOrganization">
    <lineChart :data = 'lineData'></lineChart>
    <br>
    <br>
    <treeMap :data = 'treeMapData'></treeMap>
    <br>
    <Histogram :data = 'histogramData'></Histogram>
  </div>
</template>

<script>
import lineChart from './components/Line.vue'
import treeMap from './components/Treemap.vue'
import Histogram from './components/HistogramNew.vue'
import histoData from './assets/data.js'
import allTreeData from './assets/allTreeData.js'
import lineData from './assets/lineData.js'

export default {
  name: 'app',
  components: {
    lineChart,
    treeMap,
    Histogram
  },

  created(){
    this.lineData = lineData
    this.treeMapData = allTreeData
    this.histogramData = histoData
  },

  methods: {},
  data() {
    return {lineData :[], treeMapData: [], histogramData: []}
  }
}
</script>

<style>

</style>