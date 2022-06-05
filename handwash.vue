<template>
  <q-page class="flex flex-center">


    <q-dialog v-model="detected" persistent>
      <q-card>
        <q-card-section class="row items-center">
          <q-avatar icon="thumb_up" color="primary" text-color="white" />
          <span class="q-ml-sm">Update Success</span>
        </q-card-section>

        <q-card-actions align="right">
          <q-btn flat label="OK" color="primary" v-close-popup />
        </q-card-actions>
      </q-card>
    </q-dialog>

    <div>
    <q-btn label="Test" color="primary" @click="detected = true" />
    </div>


  <marquee width="1180px" height="100" class="text-h1" :scrollamount="marquee_speed" style="background-color:DodgerBlue;color:whitesmoke;">{{marquee_text}}</marquee>
  <!-- </div> -->

    <div class="q-pa-none q-pm-none">

              <div style="width: 1180px; height: 750px;">
                <q-media-player
                  type="video"
                  background-color="black"
                  :muted="true"
                  :autoplay="true"
                  :sources="top_video"
                  :loop="true"
                  :disabled-seek="true"
                  :hide-volume-btn="true"
                  :no-controls="true"
                ></q-media-player>

                <q-media-player
                  type="video"
                  background-color="black"
                  :muted="true"
                  :autoplay="true"
                  :sources="center_video"
                  :loop="true"
                  :disabled-seek="true"
                  :hide-volume-btn="true"
                  :no-controls="true"
                ></q-media-player>

                <q-img src="/buttom.jpg" width="1180px" height="200px" />

              </div>
    <!-- <iframe src="http://192.168.1.188:8000/foyer/14/" height="750" width="1180" style="border:none;dispaly:block；" >
    </iframe>
    <iframe src="http://192.168.1.188:8000/foyer/14/" height="350" width="1180" style="border:none;dispaly:block；" >
    </iframe> -->
   </div>
  </q-page>
</template>

<script>

import { Loading } from 'quasar'
import axios from 'axios';

export default {
  name: 'App',
  data () {
    return {
      detected: false,
      marquee_speed:"",
      marquee_text:"",
      TempLabel: {
        min: "",
        max: ""
      },
      // marquee_text:"目前防疫溫度偵測進行中，如體溫過高者將發出提醒畫面與聲音，請配合警衛再進一步量測溫度，如造成您的不便敬請見諒!!!***謝謝***",
      intervalId: null, //data 定義一個定時器id
      top_video: [{
          // src: 'https://ftp.nluug.nl/pub/graphics/blender/demo/movies/ToS/ToS-4k-1920.mov',
          src: '/top.mov',
          type: 'video/mp4'
        }],
      center_video: [{
          // src: 'https://ftp.nluug.nl/pub/graphics/blender/demo/movies/ToS/ToS-4k-1920.mov',
          src: '/center.mov',
          type: 'video/mp4'
        }],
      temperature: 0,

    }
  },
  created () {
    // Subscribe mqtt
    this.subscribe_topic = 'handwash/#'
    this.$mqtt.subscribe(this.subscribe_topic)


    let one = "http://192.168.1.191:8080/marquee.txt"
    let two = "http://192.168.1.191:8080/temperature.txt"

    const requestOne = axios.get(one);
    const requestTwo = axios.get(two);

    axios.all([requestOne, requestTwo]).then(axios.spread((...responses) => {

      const responseOne = responses[0].data

      this.marquee_speed = responseOne.marquee_speed
      this.marquee_text = responseOne.marquee_text

      const TempLabel = responses[1].data

      this.TempLabel.max = TempLabel.maxTemp
      this.TempLabel.min = TempLabel.minTemp

       console.log('responseOne: ', responseOne);
       console.log('TempLabel: ', TempLabel);
          // use/access the results
        })).catch(errors => {
          // react on errors.
          console.log('error ');
        })

  },
  methods:{
    lang(){
      if(this.intervalId != null ) return;

      this.intervalId = setInterval(() => {
        //得到第一個字元
        var start = this.marquee_text.substring(0,1)
       //得到最後一個字元
        var end = this.marquee_text.substring(1)
        //後面與前面字元連接
        this.marquee_text = end + start
      },100)
    },
    //停止計時器
    stop(){
      clearInterval(this.intervalId)
      //當清除定時器之後，重新讓intervalId為null
      this.intervalId = null;
    }
  },
  mounted () {

  },
  mqtt: {
    'handwash/#' (data, topic) {
      this.object = JSON.parse(data)

   //    console.log('topic: ', topic, ' this.object: ', this.object)

    }
  },

}
</script>

<style>
  div.scrollmenu {
    overflow    : auto;
    white-space : nowrap;
    margin  : 4px;
  }

  div.scrollmenu .thumbnail {
    display : inline-block;
    margin  : 4px;
  }
</style>
