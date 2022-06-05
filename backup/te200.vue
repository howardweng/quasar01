<template>
  <q-page>
    <div class="q-pa" >
      <q-carousel
        v-model="slide"
        transition-prev="slide-right"
        transition-next="slide-left"
        animated
        :fullscreen.sync="fullscreen"
        class="rounded-borders"
        style="height:auto; "
      >

        <q-carousel-slide name="overview" class="column no-wrap" >
          <div class="row justify-center q-pa">

            <b style="font-size:28px; font-family: Microsoft JhengHei; line-height: 54px;">
              冰箱溫度監控
            </b>
            <q-item  id="linechart" style="width:100%; height:100%;  min-height: 300px;"/>

          </div>
        </q-carousel-slide>

      </q-carousel>
    </div>
  </q-page>
</template>

<script>
import axios from "axios";
// import LineChart from "../components/LineChart.js";
// import PieChart from "../components/PieChart.js";
import moment from 'moment'
import {Loading, QSpinnerGears} from 'quasar'
// import ChartJspluginDoughnutlabel from "chartjs-plugin-doughnutlabel";
// import Button from "../components/button";
import Vue from "vue";
import echarts from 'echarts';

var all_device_MAC = [];
var echart_thermometer = {};
var line_chart_temperature = {};

var data_A1 = [];
var data_A2 = [];
var data_B1 = [];
var data_B2 = [];
var update_nowtime = []

var userInfo = ''

var nowtime = 0;
var min_time = 0;

export default {
  name: "PageIndex",
  components: {
    // LineChart,
    // PieChart,
    // Button
  },
  data() {
    return {
      okaudio: "",
      // all_device_MAC: [],
      slide: "overview",
      lorem: "Hi!",
      event_list: [
        {
          id: 1,
          rack_name: "mouse",
          cage_id: "3",
          event_type: "low volt"
        },
        {
          id: 2,
          rack_name: "mouse",
          cage_id: "4",
          event_type: "dead warning"
        }
      ],
      loaded: false,
      fullscreen: false,
      doughnut_alive_device_data: null,
      doughnut_battery_alert_data: null,
      doughnut_dirt_alert_data: null,
      doughnut_feeder_alert_data: null,
      chartdata: null,
      alive_device_options: {},
      battery_alert_options: {},
      dirt_alert_options: {},
      feeder_alert_options: {
        responsive: true,
        plugins: {
          doughnutlabel: {
            labels: []
          }
        }
      }
      //user_lastdata:[]
    };
  },
  computed: {

  },
  created() {
    this.okaudio = new Audio("https://mk100.frrut.net/when.mp3");
    this.loaded = false;

    Loading.show({
      delay: 400,
      spinner: QSpinnerGears
    })
    this.dashboard_temp_get_api();
    
    // this.topicmqtt = "te200/data/#";
    // this.$mqtt.subscribe(this.topicmqtt);
    // this.interval_update_time()
  },
  methods: {
    playOKSound() {
      console.log("Play OK sound");
      const okaudioPromise = this.okaudio.play();
      if (okaudioPromise !== undefined) {
        okaudioPromise
          .then(() => {
            console.log("Finished Play OK sound");
            // Automatic playback started!
            // Show playing UI.
          })
          .catch(error => {
            // Auto-play was prevented
            // Show paused UI.
          });
      }
    },
    checkTime() {
      return Date.parse(new Date());
    },
    dashboard_temp_get_api() {

      // const requestOne = axios.get("https://api.frrut.net/device-collections");
      const requestTwo = axios.get("https://api.frrut.net/te-200-last-data");
      const requestThree = axios.get("https://api.frrut.net/mk-100-users");
      // const requestFour = axios.get("https://api.frrut.net/tm-200-all-data?_limit=3000&_sort=timestamp:DESC")

      min_time = moment(this.checkTime() - 3600 * 1000 * 1).locale('zh-TW').format('YYYY/MM/DD HH:mm:ss')
      nowtime = moment(this.checkTime()).locale('zh-TW').format('YYYY/MM/DD HH:mm:ss')
      
      console.log("min_time: ", min_time, typeof(min_time));
      console.log("nowtime: ", nowtime, typeof(nowtime));

      axios.all([requestTwo, requestThree])
        .then(
          axios.spread((...responses) => {
            // const deviceCollection = responses[0].data;
            const lastdata = responses[0].data.filter(function(item) {
              return item.mac;
            });
            const mk100users = responses[1].data;

            // userInfo = [JSON.parse(localStorage.getItem("loginInfo"))];
            userInfo = [
              {"TE200_MAC": ['2CF432778CC4', '2CF432778CE7'],
              "id": "5eec2a93b7808300688b63e3",
              "loginStatus": true,
              "role": "normal",
              "username": "brian_temp"}
            ]

            for (var e = 0; e < userInfo.length; e++) {
              for (var f = 0; f < mk100users.length; f++) {
                if (userInfo[e].id === mk100users[f]._id) {
                  userInfo[e].role = mk100users[f].role;
                  userInfo[e].TE200_MAC = mk100users[f].TE200_MAC
                }
              }
            }
            console.log("userInfo: ", userInfo)

            var request_te200_all_data_URL = "https://api.frrut.net/tm-200-all-data?_limit=-1&timestamp_gte=" + String(this.checkTime()-86400000)

            if(userInfo[0].TE200_MAC !== undefined){
              if(userInfo[0].TE200_MAC.length > 0){
                for(var i = 0; i < userInfo[0].TE200_MAC.length; i++){
                 request_te200_all_data_URL += "&mac=" + userInfo[0].TE200_MAC[i]
                }
                axios.get(request_te200_all_data_URL)
                .then(response => {
                  console.log("len of all data: ", response.data.length)
                  
                  for (var j = 0; j < response.data.length; j++) {
                    if(response.data[j].mac === userInfo[0].TE200_MAC[0]){
                      data_A1.push({name:response.data[j].mac, value:[response.data[j].timeStr, response.data[j].temp01]})
                      data_A2.push({name:response.data[j].mac, value:[response.data[j].timeStr, response.data[j].temp02]})
                    }
                    else if(response.data[j].mac === userInfo[0].TE200_MAC[1]){
                      data_B1.push({name:response.data[j].mac, value:[response.data[j].timeStr, response.data[j].temp01]})
                      data_B2.push({name:response.data[j].mac, value:[response.data[j].timeStr, response.data[j].temp02]})
                    }
                  }

                  update_nowtime.push({value:[moment(this.checkTime()).locale('zh-TW').format('YYYY/MM/DD HH:mm:ss'), -1000]})
                  
                  console.log("last_data_for_mqtt: ", lastdata)

                  this.initialize_temperature()
                  this.interval_update_time()
                  this.topicmqtt = "te200/data/#";
                  this.$mqtt.subscribe(this.topicmqtt);

                  Loading.hide()
                })
                .catch(error => {
                  console.log(error)
                })
              }
              else{
                Loading.hide()
              }
              
            }
            else{
              Loading.hide()
            }
          })
        )
        .catch(e => {
          console.error(e);
        });
    },
    interval_update_time() {
      setInterval(() => {
        console.log("Update time!!!")
        min_time = moment(this.checkTime() - 3600 * 1000 * 1).locale('zh-TW').format('YYYY/MM/DD HH:mm:ss')
        update_nowtime = []
        update_nowtime.push({value:[moment(this.checkTime()).locale('zh-TW').format('YYYY/MM/DD HH:mm:ss'), -1000]})
        update_nowtime.shift()

        line_chart_temperature.setOption({
          xAxis: {
            min: min_time
          },
          series: [{
            name: '冰箱溫度:A1',
            data: data_A1,
          },
          {
            name: '冰箱溫度:A2',
            data: data_A2,
          },
          {
            name: '冰箱溫度:B1',
            data: data_B1,
          },
          {
            name: '冰箱溫度:B2',
            data: data_B2,
          },
          {
            name: 'update_time',
            data: update_nowtime,
          },
          ]
          });
      }, 15000);
    },
    initialize_temperature(){

      var option = {
        color: ['#00A1FF', '#00FFF5', "#8800FF", '#C480FF'],
        grid: {
            bottom: 60,
        },
        // title: {
        //     text: '疫苗冰箱溫度監控',
        //     x: 'center',
        //     textStyle: {
        //         fontWeight: 'bold',
        //         fontSize: 20
        //     }
        // },
        tooltip: {
            trigger: 'axis',
            confine: true,
            textStyle: {
                fontSize: 14
            }
        },
        xAxis: {
            min: min_time,
            // min: yesterday_time,
            // max: nowtime,
            name: "時間",
            nameLocation: 'center',
            nameTextStyle:{
              fontWeight: "bold",
              fontSize: 10,
              // padding: [0, 300, 0, 0],
            },
            nameGap: 100,
            // maxInterval: 3600 * 1000 * 0.5, //one_day
            maxInterval: 30 * 1000, //one_day
            boundaryGap: ['200%', '200%'],
            type: "time",
            boundaryGap: false,
            axisLabel: {
                formatter: (function(value){
                    return moment(value).format('mm');
                }),
                interval: 'auto',
            },
        },
        yAxis: {
            name:"溫度°C",
            nameTextStyle:{
              fontWeight: "bold",
              fontSize: 16,
            },
            type: "value",
            max: 12,
            min: 0,
            interval: 2,
        },
        series: [
          {
            name: "冰箱溫度:A1",
            type: "line",
            lineStyle: {
              width: 4,
              color: "#00A1FF"
            },
            data: data_A1,
            showSymbol: false,
            hoverAnimation: false,
            smooth: true,
            symbolSize: [10, 10],
            animationDuration: 1600,
          }, 
          {
            name: "冰箱溫度:A2",
            type: "line",
            lineStyle: {
              width: 4,
              color: "#00FFF5"
            },
            data: data_A2,
            showSymbol: false,
            hoverAnimation: false,
            smooth: true,
            symbolSize: [10, 10],
            animationDuration: 1400,
          },
          {
            name: "冰箱溫度:B1",
            type: "line",
            lineStyle: {
              width: 4,
              color: "#8800FF"
            },
            data: data_B1,
            showSymbol: false,
            hoverAnimation: false,
            smooth: true,
            symbolSize: [10, 10],
            animationDuration: 1800
          }, 
          {
            name: "冰箱溫度:B2",
            type: "line",
            lineStyle: {
              width: 4,
              color: "#C480FF"
            },
            data: data_B2,
            showSymbol: false,
            hoverAnimation: false,
            smooth: true,
            symbolSize: [10, 10],
            animationDuration: 1500
          },
          {
            type: "line",
            data:update_nowtime,
            symbolSize:0,
            label:{
              show:false
            },
            tooltip:{
              show:false
            }
          },
          {
            name: "警戒線",
            type: "line",
            markLine: {
                lineStyle: {
                    width: 3,
                    type:"solid"
                },
                label: {
                    position: "insideEndTop",
                    formatter: "{b}",
                    color: "#333",
                    fontSize: 14
                },
                tooltip: {
                    trigger: 'item',
                    confine: true,
                    textStyle: {
                        fontSize: 14
                    },
                },
                symbolSize: 0,
                data: [{
                    name: "高溫警戒",
                    yAxis: 8.5,
                    lineStyle: {
                      color: "#f60902"
                    }
                }, {
                    name: "中警戒",
                    yAxis: 7.5,
                    lineStyle: {
                      color: "#A3A3A3"
                    }
                }]
            },
          }
        ]
      };

      line_chart_temperature.setOption(option)
    },

  },
  mounted() {
    line_chart_temperature = echarts.init(document.getElementById('linechart'))
    window.addEventListener("resize", line_chart_temperature.resize);
  },

  mqtt: {
    "te200/data/#"(mqtt_data_payload, topic) {
      this.obj = JSON.parse(mqtt_data_payload);
      console.log("this.obj: ", this.obj);
      var checkMAC = this.obj.mac  
      if(checkMAC === userInfo[0].TE200_MAC[0]){
        data_A1.push({name:this.obj.mac, value:[this.obj.timeStr, this.obj.temp01]})
        data_A2.push({name:this.obj.mac, value:[this.obj.timeStr, this.obj.temp02]})
        data_A1.shift()
        data_A2.shift()
        // this.playOKSound()
      }
      if (checkMAC === userInfo[0].TE200_MAC[1]) {
        data_B1.push({name:this.obj.mac, value:[this.obj.timeStr, this.obj.temp01]})
        data_B2.push({name:this.obj.mac, value:[this.obj.timeStr, this.obj.temp02]})
        data_B1.shift()
        data_B2.shift()
        // this.playOKSound()
      }
      line_chart_temperature.setOption({
        series: [{
          name: '冰箱溫度:A1',
          data: data_A1,
        },
        {
          name: '冰箱溫度:A2',
          data: data_A2,
        },
        {
          name: '冰箱溫度:B1',
          data: data_B1,
        },
        {
          name: '冰箱溫度:B2',
          data: data_B2,
        }]
      });
    } 
  }
};
</script>

<style lang="scss">
.background-image {
  height: 100%;
  width: 50%;
  z-index: -1;
  background-position: left center;
  background-repeat: no-repeat;
  opacity: 0.6;
}
.inner {
  // background-color: #d5e1a3;
  display: flex;           /* ★ */
  height: 300px;
  align-items: center;    /* ★ */
}

</style>
<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
