<template>
  <q-page class="flex flex-center">
<template>
  <div class="q-pa-none q-pm-none">
        <div class="scrollmenu">
          <div v-for="item in imglist"
               class="thumbnail">
            <img :src="item.url" width="130px">
            <br>
            <!-- <q-badge color="green" size="lg" rounded :label="item.temperature"/> -->
            <q-btn padding="5px 35px" :color="item.color" size="20px"  :label="item.temperature" />
          </div>
         </div>
      </div>
    <div class="q-pa-none q-pm-none">
    <!-- <iframe src="src/assets/1.mp4" height="750" width="1130" style="border:none;dispaly:block；" >
    </iframe> -->

              <div style="width: 1180px; height: 750px;">
                <q-media-player
                  type="video"
                  background-color="black"
                  :muted="true"
                  :autoplay="true"
                  :sources="sources"
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
                  :sources="sources2"
                  :loop="true"
                  :disabled-seek="true"
                  :hide-volume-btn="true"
                  :no-controls="true"
                ></q-media-player>
              </div>
    <!-- <iframe src="http://192.168.1.188:8000/foyer/14/" height="750" width="1180" style="border:none;dispaly:block；" >
    </iframe>
    <iframe src="http://192.168.1.188:8000/foyer/14/" height="350" width="1180" style="border:none;dispaly:block；" >
    </iframe> -->
   </div>
</template>
  </q-page>
</template>

<script>

import { Loading } from 'quasar'

export default {
  name: 'App',
  data () {
    return {
      sources: [{
          // src: 'https://ftp.nluug.nl/pub/graphics/blender/demo/movies/ToS/ToS-4k-1920.mov',
          src: '/2.mov',
          type: 'video/mp4'
        }],
      sources2: [{
          // src: 'https://ftp.nluug.nl/pub/graphics/blender/demo/movies/ToS/ToS-4k-1920.mov',
          src: '/1.mov',
          type: 'video/mp4'
        }],
      temperature: 0,
      sample_data : [{'img_url': 'https://cdn.quasar.dev/img/parallax2.jpg', 'temperature': 39.5}],
      //imglist : [{'url': 'https://cdn.quasar.dev/img/parallax2.jpg', 'temperature': '39.5°C'},{'url': 'https://cdn.quasar.dev/img/parallax2.jpg', 'temperature':'39.5°C'},{'url': 'https://cdn.quasar.dev/img/parallax2.jpg', 'temperature': '39.5°C'},{'url': 'https://cdn.quasar.dev/img/parallax2.jpg', 'temperature': '39.5°C'},{'url': 'https://cdn.quasar.dev/img/parallax2.jpg', 'temperature': '39.5°C'},{'url': 'https://cdn.quasar.dev/img/parallax2.jpg', 'temperature': '39.5°C'},{'url': 'https://cdn.quasar.dev/img/parallax2.jpg', 'temperature': '39.5°C'}]
      imglist:  [{'url': 'https://cdn.quasar.dev/img/parallax2.jpg',
                  'temperature': '37.5°C',
                  'color': 'grey-6'   
                }, {
                  'url': 'https://cdn.quasar.dev/img/parallax2.jpg',
                  'temperature': '37.5°C',
                  'color': 'grey-6'
                }, {
                  'url': 'https://cdn.quasar.dev/img/parallax2.jpg',
                  'temperature': '39.5°C',
                  'color': 'negative'
                }, {
                  'url': 'https://cdn.quasar.dev/img/parallax2.jpg',
                  'temperature': '37.3°C',
                  'color': 'grey-6'
                }, {
                  'url': 'https://cdn.quasar.dev/img/parallax2.jpg',
                  'temperature': '36.1°C',
                  'color': 'grey-6'
                }, {
                  'url': 'https://cdn.quasar.dev/img/parallax2.jpg',
                  'temperature': '35.2°C',
                  'color': 'grey-6'
                }, {
                  'url': 'https://cdn.quasar.dev/img/parallax2.jpg',
                  'temperature': '36.9°C',
                  'color': 'grey-6'
                }]
    }
  },
  created () {
    // Subscribe mqtt
    this.subscribe_topic = 'handwash/#'
    this.$mqtt.subscribe(this.subscribe_topic)

    this.first_initial().then(
      init_result => {
        // console.log('init_result: ', init_result)
        this.created_publish = 'created'
        this.$mqtt.publish('handwash/created', this.created_publish)
      }
    )
  },
  mounted () {

  },
  mqtt: {
    'handwash/#' (data, topic) {
      this.object = JSON.parse(data)

      console.log('topic: ', topic, ' this.object: ', this.object)
      if (topic === 'handwash/created') {
        // 【Initialization】End
        if (this.object.result === 'legal') {
          // this.preload(2001, 2128).then(
          //   result => {
          //     setTimeout(() => {
          //       Loading.hide()
          //       this.loading_finish = true
          //       // console.log("preload result: ", result)
          //     }, 5000)
          //   }
          // )
          setTimeout(() => {
            Loading.hide()
            this.loading_finish = true
          }, 3000)
        }
        else {
          this.loading_message = "<b>此裝置未註冊，無法顯示</b>"

        }
      }
      else if (topic === 'handwash/command/test'){
        this.object = JSON.parse(data)

        console.log('topic: ', topic, ' this.object: ', this.object)

        this.temperature = this.object.temperature

        console.log(' this.temperature: ', this.temperature)

        if(this.processing === false && this.loading_finish === true){
          // this.process(this.object.command)

          this.process_done_publish(this.object.command).then(
            process_result => {
              // console.log('process_result: ', process_result)

              // if(process_result === true){
              //   this.process_publish = this.object.command + '/done'
              //   this.$mqtt.publish('quasar/process_done', this.process_publish)
              // }
              // else{
              //   this.process_publish = this.object.command + '/fail'
              //   this.$mqtt.publish('quasar/process_done', this.process_publish)
              // }
            }
          )
        }
      }
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
