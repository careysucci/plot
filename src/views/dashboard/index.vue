<template>
  <div class="dashboard-container">
    <el-form :inline="true" :model="dataForm" class="demo-form-inline">
      <el-form-item label="设备">
        <el-select v-model="deviceName" placeholder="请选择">
          <el-option
            v-for="item in dataForm.deviceList"
            :key="item.value"
            :label="item.label"
            :value="item.value">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="应用">
        <el-select v-model="packageName" placeholder="请选择">
          <el-option
            v-for="item in dataForm.appList"
            :key="item.value"
            :label="item.label"
            :value="item.value">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item>
        <el-button :type="config.startbtnType">{{config.label}}</el-button>
      </el-form-item>
    </el-form>
    <div class="main-content" style="height: 100%;" id="mem">
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
    data() {
      return {
        deviceName: '',
        packageName: '',
        data: [],
        date: [],
        echart: null,
        config: {
          startbtnType: 'success',
          label: '开始'
        },
        dataForm: {
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
        },

        option: {
          tooltip: {
            trigger: 'axis'
          },
          legend: {
            data: ['pss', 'native']
          },
          grid: {
            bottom: '30%'
          },
          dataZoom: [{
            type: 'inside',
            start: 0,
            end: 10
          }, {
            start: 0,
            end: 10,
            handleIcon: 'M10.7,11.9v-1.3H9.3v1.3c-4.9,0.3-8.8,4.4-8.8,9.4c0,5,3.9,9.1,8.8,9.4v1.3h1.3v-1.3c4.9-0.3,8.8-4.4,8.8-9.4C19.5,16.3,15.6,12.2,10.7,11.9z M13.3,24.4H6.7V23h6.6V24.4z M13.3,19.6H6.7v-1.4h6.6V19.6z',
            handleSize: '80%',
            handleStyle: {
              color: '#fff',
              shadowBlur: 3,
              shadowColor: 'rgba(0, 0, 0, 0.6)',
              shadowOffsetX: 2,
              shadowOffsetY: 2
            }
          }],
          xAxis: {
            type: 'time',
            splitLine: {
              show: false
            },
          },
          yAxis: {
            type: 'value',
            boundaryGap: [0, '100%'],
            splitLine: {
              show: false
            }
          },
          series: [
            {
              name: 'pss',
              type: 'line',
              showSymbol: false,
              hoverAnimation: false,
              data: []
            },
            {
              name: 'native',
              type: 'line',
              showSymbol: false,
              hoverAnimation: false,
              data: []
            }
          ]
        }
      }
    },
    methods: {
      initChart() {
        if (this.echart) {
          this.echart.setOption(this.option)
        } else {
          this.echart = echarts.init(document.getElementById('mem'))
          this.echart.setOption(this.option)
        }
      },
      updateData() {

      },
      resize() {
        return this.echart ? this.echart.resize() : ''
      },
      randomData() {
        let now = +new Date('1907')
        let oneDay = 24 * 3600 * 1000;
        let value = Math.random() * 10000;
        now = new Date();
        value = value + Math.random() * 21 - 10;
        this.data.push([
          {
            value: [[now.getFullYear(), now.getMonth() + 1, now.getDate()].join('/')
            + ' ' +
            [now.getHours(), now.getMinutes(), now.getSeconds()].join(':'),
              Math.round(value)]
          },
          {
            value: [[now.getFullYear(), now.getMonth() + 1, now.getDate()].join('/')
            + ' ' +
            [now.getHours(), now.getMinutes(), now.getSeconds()].join(':'),
              Math.round(value)* Math.random()]
          }
        ]);
      },
      generateData(){
          return new Promise(((resolve, reject) => {
            if (this.data.length === 1000){
              this.data.shift()
            }
            this.randomData()

            this.echart.setOption({
              series:[
                {
                  data: this.data.map(item => item[0])
                },
                {
                  data: this.data.map(item => item[1])
                }
              ]
            })
            resolve()
          })).catch(error => reject(error))
      }
    },
    created() {
      let ws = new WebSocket('ws://127.0.0.1:8000/')

      ws.onmessage = function (event) {
        console.log('event: ' + event)
      }
      ws.onopen = function (event) {
        console.log('event: ' + event)
      }
    },
    watch: {
      data: {
        handler: function () {
          // this.initChart()
        },
        deep: true
      }
    },
    mounted() {
      this.randomData()
      this.initChart()
      window.addEventListener('resize', this.resize)
      this.timer = setInterval(this.generateData, 1000)
    },
    destroyed() {
      window.removeEventListener('resize', this.resize)
      clearInterval(this.timer)
    }
  }
</script>

<style lang="scss" scoped>
  .dashboard {
    &-container {
      margin: 30px;
      position: absolute;
      top: 10px;
      bottom: 0;
      left: 10px;
      width: 100%;
    }

    &-text {
      font-size: 30px;
      line-height: 46px;
    }
    .main-content {
      margin-left: 30px;
    }
  }
</style>
