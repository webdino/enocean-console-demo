<template>
  <div>
    <div class="Chart">
      <chart :chartData="chartData" :timeUnit="timeUnit" :xAxesMax="xAxesMax" :xAxesMin="xAxesMin" :height="height"></chart>
    </div>
  </div>
</template>

<script>
import { mapGetters } from 'vuex'
import Chart from './Chart'
import { isInRange, getxAxesMin, reduceArray } from '@/js/utils.js'

export default {
  components: {
    Chart
  },
  props: ['range', 'timeUnit', 'height'],
  computed: {
    chartData () {
      // storeのデータが空のとき, 空のChartデータを返す
      if (this.humidArray.length === 0) {
        return {
          datasets: [
            {
              label: 'anomaly score',
              yAxisID: 'y-axis-2',
              backgroundColor: 'rgba(255, 0, 0, 0.5)'
            },
            {
              label: '湿度',
              yAxisID: 'y-axis-1',
              backgroundColor: '#BAFF91'
            }
          ]
        }
      }
      return this.setHumidData()
    },
    xAxesMax () {
      if (this.humidArray.length === 0) {
        return null
      }
      return this.range === 'all' ? null : new Date(this.humidArray[this.humidArray.length - 1].timestamp)
    },
    xAxesMin () {
      if (this.humidArray.length === 0) {
        return null
      }
      return getxAxesMin(this.humidArray, this.range)
    },
    ...mapGetters([
      'humidArray'
    ])
  },
  methods: {
    setHumidData () {
      let humidArrayRanged = this.humidArray.filter((o) => isInRange(this.range, new Date(o.timestamp), new Date(this.humidArray[this.humidArray.length - 1].timestamp)))
      let humidDataArray = humidArrayRanged.map((o) => o.data)
      this.$emit('calculated', {
        max: Math.max.apply(null, humidDataArray.map((v) => Math.round(v * 100) / 100)),
        min: Math.min.apply(null, humidDataArray.map((v) => Math.round(v * 100) / 100)),
        ave: Math.round(humidDataArray.reduce((sum, value) => sum + value) / humidDataArray.length * 100) / 100
      })

      let humidArrayRangedReduced = reduceArray(humidArrayRanged, this.range)
      return {
        datasets: [
          {
            label: 'anomaly score',
            yAxisID: 'y-axis-2',
            backgroundColor: 'rgba(255, 0, 0, 0.5)',
            data: humidArrayRangedReduced.map((o) => { return { x: new Date(o.timestamp), y: o.ret } })
          },
          {
            label: '湿度',
            yAxisID: 'y-axis-1',
            backgroundColor: '#BAFF91',
            data: humidArrayRangedReduced.map((o) => { return { x: new Date(o.timestamp), y: o.y } })
          }
        ]
      }
    }
  }
}
</script>

<style scoped>
.placeholders {
  padding-bottom: 0.5rem;
}
.buttons {
  text-align: center;
}
</style>
