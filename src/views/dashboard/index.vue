<template>
  <div class="dashboard-container">
    <el-form :inline="true" :model="dataForm" class="demo-form-inline">
      <el-form-item label="设备">
        <el-select v-model="dataForm.deviceName" placeholder="请选择">
          <el-option
            v-for="item in dataForm.deviceList"
            :key="item.value"
            :label="item.label"
            :value="item.value">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="应用">
        <el-select v-model="dataForm.packageName" placeholder="请选择">
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
    <div class="main-content" style="height: 50%; width: auto" id="mem">
      echart
    </div>
    <div class="main-content" style="height: 50%; width: auto" id="cpu">
      echart
    </div>
  </div>
</template>

<script>
  import echarts from 'echarts'
  import moment from 'moment'

  export default {
    name: 'Dashboard',
    computed: {
    },
    data() {
      return {
        data: [],
        date: [],
        ws: null,
        echart: null,
        echartA: null,
        callback: [],
        config: {
          startbtnType: 'success',
          label: '开始'
        },
        dataForm: {
          deviceList: [],
          appList: [],
          packageName: '',
          deviceName: '',
          ip: 'ws://10.192.126.32:9570/',
        },
        option: {
          tooltip: {
            trigger: 'axis'
          },
          legend: {
            data: ['pss', 'native']
          },
          grid: {
            top: '10px'
          },
          toolbox: {
            show: true,
            right: '20%',
            feature: {
              saveAsImage: {}
            }
          },
          dataZoom: [{
            type: 'slider',
            show: true,
            start: 50,
            end: 100
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
              markLine: {
                data: [
                  {type: 'average', name: '平均值'}
                ]
              },
              data: []
            },
            {
              name: 'native',
              type: 'line',
              showSymbol: false,
              hoverAnimation: false,
              markLine: {
                data: [
                  {type: 'average', name: '平均值'}
                ]
              },
              data: []
            }
          ]
        },
        optionA: {
          tooltip: {
            trigger: 'axis'
          },
          legend: {
            data: ['cpu', 'native']
          },
          grid: {
            top: '20px'
          },
          toolbox: {
            show: true,
            right: '20%',
            feature: {
              saveAsImage: {}
            }
          },
          dataZoom: [{
            type: 'slider',
            show: true,
            start: 50,
            end: 100
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
              name: 'cpu',
              type: 'line',
              showSymbol: false,
              hoverAnimation: false,
              markLine: {
                data: [
                  {type: 'average', name: '平均值'}
                ]
              },
              data: []
            },
            {
              name: 'native',
              type: 'line',
              showSymbol: false,
              hoverAnimation: false,
              markLine: {
                data: [
                  {type: 'average', name: '平均值'}
                ]
              },
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
        if (this.echartA){
          this.echartA.setOption(this.optionA)
        } else {
          this.echartA = echarts.init(document.getElementById('cpu'))
          this.echartA.setOption(this.optionA)
        }
      },
      updateData() {

      },
      resize() {
        return this.echart ? this.echart.resize() : ''
      },
      randomData() {
        let now = (new Date()).toLocaleTimeString().replace(/^\D*/, '');
        let oneDay = 24 * 3600 * 1000;
        let value = Math.random() * 10000;
        now = moment(new Date()).format('YYYY-MM-DD HH:mm:ss');
        value = value + Math.random() * 21 - 10;
        this.data.push([
          {
            name: now.toString(),
            value: [now,
              Math.round(value)]
          },
          {
            name: now.toString(),
            value: [now,
              Math.round(value)* Math.random()]
          },
        ]);
      },
      generateData(){
        if (this.data.length === 1000){
          this.data.shift()
        }
        this.randomData()
        this.echart.setOption({
          series: [
            {
              data: this.data.map(item => item[0])
            },
            {
              data: this.data.map(item => item[1])
            }
          ]
        })
      },
      initWebSocket(){
        return new Promise((resolve, reject) => {
          this.ws = new WebSocket(`ws://${this.dataForm.ip}/`)
          resolve()
        }).then(() => {
          this.ws.onopen(event => {
            console.log("connected.")
            this.getDeviceInfo(data)
          })

          this.ws.onmessage(event => {
            console.log('onmessage.')
            const data = JSON.parse(event.data)
            const method = this.callback.pop()
            console.log(method)
            method(data)
          })
          this.ws.onclose(event => {
            this.ws.close()
          })
        }).catch(error => {
          console.log(error)
            if (this.ws){
              this.ws.close()
            }
            reject(error)
          })
      },
      initDeviceData(res){
        let data = res.devs
        if (data) {
          data.forEach(item => {
            this.dataForm.deviceList.push({
              label: item,
              value: item
            })
          })
        }
      },
      getDeviceInfo(){
        return new Promise((resolve, reject) => {
          this.ws.send('getdevinfo')
          this.callback.push(this.initDeviceData)
          resolve()
        }).catch(error => {
          console.log(error)
          reject(error)
        })
      },
      initAppData(res){
        let data = res[this.dataForm.deviceName]
        if (data){
          data.forEach(item => {
            this.dataForm.appList.push({
              label: item,
              value: item
            })
          })
        }
      },
      getAppList(){
        return new Promise(((resolve, reject) => {
          this.ws.send(`devselect_${this.dataForm.deviceName}`)
          this.callback.push(this.initAppData)
          resolve()
        })).catch(error => {
          reject(error)
        })
      }
    },
    created() {
      this.ws = new WebSocket('ws://10.192.126.32:9570/')

      this.ws.onmessage = event => {
        const data = JSON.parse(event.data)
        const method = this.callback.pop()
        if (method instanceof Function && data){
          console.log(data)
          console.log(method)
          method(data)
        }
      }

      this.ws.onopen = event => {
        this.getDeviceInfo()
      }

      this.ws.onclose = event => {
        this.ws.close()
      }
    },
    watch: {
      data: {
        handler: function () {
          // this.initChart()
        },
        deep: true
      },
      "dataForm.deviceName": {
        handler: function () {
          console.log("getAppList")
          this.getAppList()
        },
        deep: true
      },
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
      top: 0;
      bottom: 30px;
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
