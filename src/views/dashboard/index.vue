<template>
  <div class="dashboard-container">
    <div class="header-content">
      设备:
      <el-select v-model="value" placeholder="请选择">
        <el-option
          v-for="item in deviceList"
          :key="item.value"
          :label="item.label"
          :value="item.value">
        </el-option>
      </el-select>
      应用:
      <el-select v-model="value" placeholder="请选择">
        <el-option
          v-for="item in appList"
          :key="item.value"
          :label="item.label"
          :value="item.value">
        </el-option>
      </el-select>
      <el-button :type="config.startbtnType">{{config.label}}</el-button>
    </div>
    <div class="main-content" style="height: 100%" id="mem">
      echart
    </div>
  </div>
</template>

<script>
// import { mapGetters } from 'vuex'
import echarts from 'echarts'

export default {
  name: 'Dashboard',
  computed: {
    // ...mapGetters([
    //   'name'
    // ])
  },
  data(){
    return {
      data: [],
      echart: null,
      config: {
        startbtnType: 'success',
        label: '开始'
      },
      deviceList: [
        {
          label: 'D21b',
          value: 'sdf1232'
        },
        {
          label: 'E28',
          value: '123de333'
        }
      ],
      appList: [
        {
          label: '音乐',
          value: 'com.xiaopeng.musicradio'
        },
        {
          label: '收音机',
          value: 'com.xiaopeng.dab'
        }
      ],
      option: {
        xAxis: {
          type: 'time',
          splitLine: {
            show: false
          }
        },
        yAxis: {
          type: 'value',
          boundaryGap: [0, '100%'],
          splitLine: {
            show: false
          }
        },
        series: [{
          name: '模拟数据',
          type: 'line',
          showSymbol: false,
          hoverAnimation: false,
          data: this.data
        }]
      }
    }
  },
  methods: {
    initChart(){
      if (this.echart) {
        this.echart.setOption(this.option)
      }else {
        this.echart = echarts.init(document.getElementById('mem'))
        this.echart.setOption(this.option)
      }
    },
    updateData(){

    },
    resize(){
      return this.echart ? this.echart.resize() : ''
    },
    generateData(){
      let data = [];
      let now = +new Date(1997, 9, 3);
      let oneDay = 24 * 3600 * 1000;
      let value = Math.random() * 1000;
      now = new Date(+now + oneDay);
      value = value + Math.random() * 21 - 10;
      this.data.push({
        value: [
          [now.getFullYear(), now.getMonth() + 1, now.getDate()].join('/'),
          Math.round(value)
        ]
      });
    },
  },
  created(){
    // let ws = new WebSocket('ws://127.0.0.1:8000/')
    //
    // ws.onmessage = function (event) {
    //   console.log('event: ' + event)
    // }
    // ws.onopen = function (event) {
    //   console.log('event: ' + event)
    // }
  },
  watch: {
    data: {
      handler: function () {
        this.initChart()
      },
      deep: true
    }
  },
  mounted() {
    this.generateData()
    window.addEventListener('resize', this.resize)
  },
  destroyed() {
    window.removeEventListener('resize', this.resize)
  }
}
</script>

<style lang="scss" scoped>
.dashboard {
  &-container {
    margin: 30px;
  }
  &-text {
    font-size: 30px;
    line-height: 46px;
  }
}
</style>
