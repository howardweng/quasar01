<template>
  <q-page class="flex flex-center">
<template>
  <div class="q-pa-none row items-start q-gutter-md" style="width: 100%; transform:rotate(0deg); full-width">
    <q-card flat>
    <div class="q-col-gutter-md row items-start">
        <div class="scrollmenu">
          <div v-for="item in imglist"
               class="thumbnail">
            <img :src="item.url" width="130px">
            <br>
            <!-- <q-badge color="green" size="lg" rounded :label="item.temperature"/> -->
            <q-btn padding="5px 35px" :color="item.color" size="20px"  :label="item.temperature" />
          </div>
         </div>
      <!-- <q-separator /> -->
    <iframe src="http://192.168.1.188:8000/foyer/14/" height="650" width="1040" style="border:none;" >
    </iframe>
    <iframe src="http://192.168.1.188:8000/foyer/14/" height="650" width="1040" style="border:none;" >
    </iframe>
    <iframe src="http://192.168.1.188:8000/foyer/14/" height="250" width="1040" style="border:none;" >
    </iframe>
        <!-- <img
          basic
          no-default-spinner
          class="col-12"
          style="width: 100%;"
          :src="picture"
        /> -->
      <!-- <div v-if="!processing" class="left_text" :style="image_span_style">{{ position_text_left }}</div>
        <div v-if="!processing" class="center_text" :style="image_span_style">{{ position_text }}</div> -->
      <div v-if="!processing" class="right_text" :style="image_span_style">{{ position_text_right }}</div>
      </div>
    </q-card>
  </div>
</template>
  </q-page>
</template>

<script>

import { Loading } from 'quasar'

export default {
  name: 'cannon',
  data () {
    return {
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
  }

  div.scrollmenu .thumbnail {
    display : inline-block;
    margin  : 5px;
  }
</style>
