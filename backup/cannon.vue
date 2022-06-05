<template >
  
  <div v-if="loading_finish" class="row" style="background-color: black;">
    <q-card flat class="row col-12" style="width: 70%; height:455px; background-color: white;"></q-card>
    
    <q-card flat class="row" style="width: 70%; transform:rotate(180deg);">
      <div class="col-12 container">
        <img
          basic
          no-default-spinner
          class="col-12"
          style="width: 100%;"
          :src="picture"
        />
	      <div v-if="!processing" class="left_text" :style="image_span_style">{{ position_text_left }}</div>
        <div v-if="!processing" class="center_text" :style="image_span_style">{{ position_text }}</div>
	      <div v-if="!processing" class="right_text" :style="image_span_style">{{ position_text_right }}</div>
      </div>
    </q-card>

    <q-card flat class="row col-12" style="width: 70%; height:200px; background-color: white;"></q-card>
  </div>
</template>

<script>

const redWhite_start = 1
const redWhite_end = 149

const whiteRed_start = 5829
const whiteRed_end = 5971

const red_start = 5972
const red_end = 6400

function between(x, min, max) {
  return max >= x && x >= min;
}

import {Loading} from 'quasar'

export default {
  name: 'cannon',
  data () {
    return {
      diff: 0,
      image_span_style: 'color: black;',
      
      position: '',
      direction: '',
      speed: '',
      loading_finish: false,
      picture: null,
      process_position: 0,
      original_position: 128,
      target_position: 128,
      position_text: 6400,
      position_text_left: 0,
      position_text_right: 0,

      processing: false,
      list: [],
      vague_list: [1, 2, 3, 4, 5],
      vague_list_one: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10],

      loading_message: "<b>系統載入中...</b>",
    }
  },
  created () {
     
    // 【Initialization】Start
    Loading.show({
      message: this.loading_message,
      customClass: "my-class"
    })

    // Subscribe mqtt
    this.subscribe_topic = 'cannon/#'
    this.$mqtt.subscribe(this.subscribe_topic)

    this.first_initial().then(
      init_result => {
        // console.log('init_result: ', init_result)

        this.created_publish = 'created'
        this.$mqtt.publish('quasar/created', this.created_publish)

      }
    )
  },
  mounted () {

  },
  mqtt: {
    'cannon/#' (data, topic) {
      this.object = JSON.parse(data)

      // console.log("topic: ", topic, " this.object: ", this.object)

      if(topic === 'cannon/created'){
        // 【Initialization】End
        if(this.object.result === 'legal'){
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
        else{
          this.loading_message = "<b>此裝置未註冊，無法顯示</b>"

          setTimeout(() => {
            Loading.show({
              spinner: null,
              message: this.loading_message,
              customClass: 'text-h5',
            })
          }, 1000)
        }
      }
      else if (topic === 'cannon/command'){
        if(this.processing === false && this.loading_finish === true){
          // this.process(this.object.command)

          this.process_done_publish(this.object.command).then(
            process_result => {
              // console.log('process_result: ', process_result)

              if(process_result === true){
                this.process_publish = this.object.command + '/done'
                this.$mqtt.publish('quasar/process_done', this.process_publish)
              }
              else{
                this.process_publish = this.object.command + '/fail'
                this.$mqtt.publish('quasar/process_done', this.process_publish)
              }
            }
          )
        }
      }

    }
  },
  methods: {
    async first_initial(){
      const init_new_ok = await this.process_new('6400');
      return [init_new_ok]
    },

    async process_done_publish(command){
      // var process_position = command.split('/')[0]
      var process_position = parseInt(command.split('/')[0])
      var process_done

      if(process_position > 6400 || process_position < 0){
        return new Promise(resolve => {
          this.processing = false
          resolve(false);
        });
      }
      else{
        process_done = await this.process_new(process_position)
      }

      return process_done
    },

    process_new(position){
      this.picture = this.switch_6400(position)

      return new Promise(resolve => {
          // console.log("Finish. original_position: ", this.original_position, "\ntarget_position: ", this.target_position)
          // console.log("-----------------")
          this.processing = false
          resolve(true);
      });
    },

    switch_6400(position){
      // position: '1' ~ '6400'
      // console.log("position: ", position)
      
      var number = parseInt(position)
      // var number = position

      if(number == 0){
        number = 6400
      }

      this.position_text = number

      if(number - 50 <= 0){
        this.position_text_left = number - 50 + 6400
      }
      else{
        this.position_text_left = number - 50
      }

      if(number + 50 > 6400){
        this.position_text_right = number + 50 - 6400
      }
      else{
        this.position_text_right = number + 50
      }

      if (between(number, redWhite_start, redWhite_end)){
        return require("../images/20211216_images/redWhite/RW-" + number + ".png")
      }
      else if (between(number, whiteRed_start, whiteRed_end)){
        var picture_number = number - 5828
        return require("../images/20211216_images/whiteRed/WR-" + picture_number + ".png")
      }
      else if (between(number, red_start, red_end)){
        var picture_number = number % 50
        return require("../images/20211216_images/red/RD-" + picture_number + ".png")
      }
      else{
        var picture_number = number % 50
        return require("../images/20211216_images/white/WH-" + picture_number + ".png")
      }
    }

    // async preload(start, end) {
    //   const preload_done = await this.preload_image(start, end)

    //   return preload_done
    // },

    // preload_image(start, end){
    //   return new Promise(resolve => {
    //     var start_number = start
    //     var end_number = end
    //     let pad = "00000"
    //     var directory

    //     for(var i = start_number; i <= end_number; i++){

    //       let directory_number = Math.ceil(i / 1000)
          
    //       if(directory_number < 10){
    //         directory = '0' + directory_number
    //       }
    //       else{
    //         directory = directory_number.toString()
    //       }

    //       let str = "" + i
    //       let position = pad.substring(0, pad.length - str.length) + str

    //       var img = new Image ()
    //       img.onload = function() {
    //       }
    //       img.src = require("../images/cannon_images/" + directory + "/cannon_" + position + ".png")
        
    //       if(i === end_number){
    //         resolve(true);
    //       }
    //     }
    //   });
    // },
  }
}
</script>


<style lang="scss">
.my-class {
  transform: rotate(180deg);
  font-size: 4em;
  top: 30%;
  right: 30%;
}

.container {
  position: relative;
  text-align: left;
}

.center_text {
  position: absolute;
  font-size: 5em;
  font-weight: bold;
  top: 35%;
  left: 50%;
  transform: translate(-50%, -50%);
}

.left_text {
  position: absolute;
  font-size: 4.5em;
  font-weight: bold;
  top: 35%;
  left: 14%;
  transform: translate(-50%, -50%);
}

.right_text {
  position: absolute;
  font-size: 4.5em;
  font-weight: bold;
  top: 35%;
  left: 86.5%;
  transform: translate(-50%, -50%);
}

</style>

