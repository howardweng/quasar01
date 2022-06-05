<template >
  
  <div v-if="loading_finish" class="row" style="background-color: black;">
    <div class="text-h2 text-red">test!!</div>
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
	      <div v-if="!processing && !new_mode" class="left_text" :style="image_span_style">{{ position_text_left }}</div>
        <div v-if="!processing && !new_mode" class="center_text" :style="image_span_style">{{ position_text }}</div>
	      <div v-if="!processing && !new_mode" class="right_text" :style="image_span_style">{{ position_text_right }}</div>
      </div>
      <!-- <div class="q-pa-md col-12 row justify-center" style="font-size:2.5rem;">
        <q-card class="q-pa-xs" flat>
          <span v-if="processing" class='col-12 row justify-center' style="font-weight:bold; color:blue;"> {{ list[original_position ]}} </span>
          <span v-if="!processing" class='col-12 row justify-center' style="font-weight:bold; color:blue;"> {{ f }} </span>
        </q-card>
      </div> -->
    </q-card>

    <q-card flat class="row col-12" style="width: 70%; height:200px; background-color: white;"></q-card>
    
    <!-- <div class="q-pa-sm row col-12 justify-center items-center">
      <q-input class="q-pa-sm col-md-2 col-sm-3" outlined v-model="position" label="位置" />/
      <q-input class="q-pa-sm col-md-2 col-sm-3" outlined v-model="direction" label="方向" />/
      <q-input class="q-pa-sm col-md-2 col-sm-3" outlined v-model="speed" label="速度" />
      <q-btn :disable="processing" class="q-pa-sm flex flex-center col-md-1 col-sm-2" style="height:75%" no-caps label="執行" @click="process(position + '/' + direction + '/' + speed)"/>
    </div> -->
  </div>
</template>

<script>


function switch_64000(position){
  // position: '00001' ~ '64000'
  // console.log("position: ", position)
  
  var number = parseInt(position)
  var directory_number = Math.ceil(number / 1000)
  var directory

  if(directory_number < 10){
    directory = '0' + directory_number
  }
  else{
    directory = directory_number.toString()
  }

  // var path = "../images/cannon_images/" + directory + "/cannon_" + position + ".png" 
  return require("../images/cannon_images/" + directory + "/cannon_" + position + ".png")

}

function switch_rule(position){
  
  // console.log("position: ", position)
  if(position % 10 === 8 || position % 10 === 9 || position % 10 === 0 || position % 10 === 1 || position % 10 === 2 ){
    return require("../images/20210928_Final/No_Text_10.png")
  }
  else if(position % 10 === 3 || position % 10 === 4 || position % 10 === 5 || position % 10 === 6 || position % 10 === 7 ){
    return require("../images/20210928_Final/No_Text_5.png")
  }
}

function switch_vague_rule(vague_list, number){
  switch(vague_list[number]){
    case vague_list[number]:
      return require("../images/20210928_Final/No_Text_vague_" + number + "_one.png")      
  }
}

function between(x, min, max) {
  return max >= x && x >= min;
}

const cockpit_cover_start = 6005
const cockpit_cover_end = 6388

const left_rear_hatch_start = 3200
const left_rear_hatch_end = 3913

const right_rear_hatch_start = 2493
const right_rear_hatch_end = 3200

const minimum_scale = 50
const split_part = 6400 / minimum_scale

import {Loading} from 'quasar'

export default {
  name: 'cannon',
  data () {
    return {
      diff: 0,
      image_span_style: '',
      
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

      new_mode: true,
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


    // 初始化圖片及資料
    // for (var i = 0; i < split_part; i++){
    //   this.list.push(i * minimum_scale)
    // }


    // if(between(this.process_position, cockpit_cover_start, cockpit_cover_end) || between(this.process_position, left_rear_hatch_start, left_rear_hatch_end) || between(this.process_position, right_rear_hatch_start, right_rear_hatch_end)){
    //   this.image_span_style = "color: white;"
    //   if(this.process_position % 10 === 8 || this.process_position % 10 === 9 || this.process_position % 10 === 0 || this.process_position % 10 === 1 || this.process_position % 10 === 2 ){
    //     this.picture = require("../images/20210928_Final/No_Text_Red_10.png")
    //   }
    //   else if(between(this.process_position % 10, 3, 7)){
    //     this.picture = require("../images/20210928_Final/No_Text_Red_5.png")
    //   }
    // }
    // else{
    //   this.image_span_style = "color: black;"
    //   this.picture = switch_rule(this.process_position)
    // } 

    this.first_initial().then(
      init_result => {
        // console.log('init_result: ', init_result)

        this.created_publish = 'created'
        this.$mqtt.publish('quasar/created', this.created_publish)

      }
    )

    // this.created_publish = 'created'
    // this.$mqtt.publish('quasar/created', this.created_publish)
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
// 修改這行修改這行修改這行修改這行修改這行修改這行修改這行修改這行修改這行修改這行修改這行修改這行
          this.preload(2001, 2128).then(
// 修改這行修改這行修改這行修改這行修改這行修改這行修改這行修改這行修改這行修改這行修改這行修改這行
            result => {
              setTimeout(() => {
                Loading.hide()
                this.loading_finish = true
                // console.log("preload result: ", result)
              }, 5000)
            }
          )
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
      // const init_1_ok = await this.process('3200/right/speed_for_init');
      // const init_2_ok = await this.process('6400/right/speed_for_init');
      // return [init_1_ok, init_2_ok]
      
      const init_new_ok = await this.process_new('64000');
      return [init_new_ok]
      
    },
    async process_done_publish(command){

      var process_position = command.split('/')[0]

      if(process_position.length === 5){
        var process_done = await this.process_new(process_position)
      }
      else{
        var process_done = await this.process(command)
      }

      // const process_done = await this.process(command)
      return process_done
    },
    async preload(start, end) {
      
      const preload_done = await this.preload_image(start, end)

      return preload_done
    },
    preload_image(start, end){
      
      return new Promise(resolve => {
        var start_number = start
        var end_number = end
        let pad = "00000"
        var directory

        for(var i = start_number; i <= end_number; i++){

          let directory_number = Math.ceil(i / 1000)
          
          if(directory_number < 10){
            directory = '0' + directory_number
          }
          else{
            directory = directory_number.toString()
          }

          let str = "" + i
          let position = pad.substring(0, pad.length - str.length) + str

          // let path = {"url": "../images/cannon_images/" + directory + "/cannon_" + position + ".png"}
          var img = new Image ()
          img.onload = function() {
              // console.log("preload ", "../images/cannon_images/" + directory + "/cannon_" + position + ".png")
          }
          img.src = require("../images/cannon_images/" + directory + "/cannon_" + position + ".png")
        
          if(i === end_number){
            resolve(true);
          }
        }
        // resolve(true);
      });
    },
    process_new(position){
      this.new_mode = true
      this.picture = switch_64000(position)

      return new Promise(resolve => {
          // console.log("Finish. original_position: ", this.original_position, "\ntarget_position: ", this.target_position)
          // console.log("-----------------")
          this.processing = false
          resolve(true);
      });
    },
    process(process_text){

      let time_interval

      this.process_position = parseInt(process_text.split('/')[0], 10)
      this.process_direction = process_text.split('/')[1].toLowerCase()
      this.process_speed = process_text.split('/')[2].toLowerCase()

      // console.log("1this.process_position: ", this.process_position, "typeof(this.process_position): ", typeof(this.process_position))

      this.target_position = Math.floor(this.process_position / minimum_scale)

      // console.log("original_position: ", this.original_position, "\ntarget_position: ", this.target_position)

      if(this.process_position === 0 || this.target_position > split_part || this.target_position < 0 || this.process_position > 6400){
        // console.log("Over range!")
        this.processing = false
        return
      }
      else{

        this.position_text = this.process_position

        if(this.process_position - 50 <= 0){
          this.position_text_left = this.process_position - 50 + 6400
        }
        else{
          this.position_text_left = this.process_position - 50
        }

        if(this.process_position + 50 > 6400){
          this.position_text_right = this.process_position + 50 - 6400
        }
        else{
          this.position_text_right = this.process_position + 50
        }

        // console.log("2this.process_position: ", this.process_position, "typeof(this.process_position): ", typeof(this.process_position))

        // 決定轉動速度
        if(this.process_speed === "sl"){
          time_interval = 100
        }
        else if(this.process_speed === "md"){
          time_interval = 50
        }
        else if(this.process_speed === "fs"){
          time_interval = 25
        }
        else if(this.process_speed === "co"){
          time_interval = 0
        }
        else if(this.process_speed === "speed_for_init"){
          time_interval = 5
        }
        else{
          time_interval = 50
        }

        // 決定轉動方向
        if(this.process_speed === "co"){

          this.original_position = this.target_position

          if(between(this.process_position, cockpit_cover_start, cockpit_cover_end) || between(this.process_position, left_rear_hatch_start, left_rear_hatch_end) || between(this.process_position, right_rear_hatch_start, right_rear_hatch_end)){
            this.image_span_style = "color: white;"
            if(this.process_position % 10 === 8 || this.process_position % 10 === 9 || this.process_position % 10 === 0 || this.process_position % 10 === 1 || this.process_position % 10 === 2 ){
              this.picture = require("../images/20210928_Final/No_Text_Red_10.png")
            }
            else if(between(this.process_position % 10, 3, 7)){
              this.picture = require("../images/20210928_Final/No_Text_Red_5.png")
            }
          }
          else{
            this.image_span_style = "color: black;"
            this.picture = switch_rule(this.process_position)
          }
        }
        else if(this.process_direction === 'left'){

          if(this.target_position > this.original_position){
            // Ex. 原本在 3000 (60)，新位置設定 4000 (80)，距離 diff 為 128 - 20 = 108 ( 60 -> 0(128) -> 80)
            this.diff = split_part - Math.abs(this.target_position - this.original_position)
          }
          else{
            this.diff = Math.abs(this.target_position - this.original_position)
          }

          this.processing = true
          for (let i = 0; i < this.diff; i+=1){
            setTimeout(() => {
              this.original_position -= 1
              
              if(this.original_position < 0){
                this.original_position += split_part
              }

              if(i !== this.diff - 1){
                this.picture = switch_vague_rule(this.vague_list_one, (i%10 + 1))
              }
              else{
                if(between(this.process_position, cockpit_cover_start, cockpit_cover_end) || between(this.process_position, left_rear_hatch_start, left_rear_hatch_end) || between(this.process_position, right_rear_hatch_start, right_rear_hatch_end)){
                  this.image_span_style = "color: white;"
                  if(this.process_position % 10 === 8 || this.process_position % 10 === 9 || this.process_position % 10 === 0 || this.process_position % 10 === 1 || this.process_position % 10 === 2 ){
                    this.picture = require("../images/20210928_Final/No_Text_Red_10.png")
                  }
                  else if(between(this.process_position % 10, 3, 7)){
                    this.picture = require("../images/20210928_Final/No_Text_Red_5.png")
                  }
                }
                else{
                  this.image_span_style = "color: black;"
                  this.picture = switch_rule(this.process_position)
                }
              }
            }, time_interval * i);
          }

        }
        else if (this.process_direction === 'right'){

          if(this.target_position < this.original_position){
              // Ex. 原本在 3000 (60)，新位置設定 2000 (40)，距離 diff 為 128 - 20 = 108 ( 60 -> 128(0) -> 40)
              this.diff = split_part - Math.abs(this.target_position - this.original_position)
          }
          else{
              this.diff = Math.abs(this.target_position - this.original_position)
          }

          // console.log("this.diff: ", this.diff)

          let reverse_count = this.diff

          this.processing = true
          for (let i = 0; i < this.diff; i+=1){
            setTimeout(() => {
              this.original_position += 1

              if(this.original_position > split_part-1){
                this.original_position -= split_part
              }

              if(i !== this.diff - 1){
                this.picture = switch_vague_rule(this.vague_list_one, (reverse_count%10 + 1))
              }
              else{
                if(between(this.process_position, cockpit_cover_start, cockpit_cover_end) || between(this.process_position, left_rear_hatch_start, left_rear_hatch_end) || between(this.process_position, right_rear_hatch_start, right_rear_hatch_end)){
                  this.image_span_style = "color: white;"
                  if(this.process_position % 10 === 8 || this.process_position % 10 === 9 || this.process_position % 10 === 0 || this.process_position % 10 === 1 || this.process_position % 10 === 2 ){
                    this.picture = require("../images/20210928_Final/No_Text_Red_10.png")
                  }
                  else if(between(this.process_position % 10, 3, 7)){
                    this.picture = require("../images/20210928_Final/No_Text_Red_5.png")
                  }
                }
                else{
                  this.image_span_style = "color: black;"
                  this.picture = switch_rule(this.process_position)
                } 
              }              
              reverse_count -= 1
            }, time_interval * i);
          }

        }
        else {
          this.new_mode = false
          this.processing = false
          return
        }

        return new Promise(resolve => {
          setTimeout(() => {
            // console.log("Finish. original_position: ", this.original_position, "\ntarget_position: ", this.target_position)
            // console.log("-----------------")
            this.new_mode = false
            this.processing = false
            resolve(true);
          }, time_interval * this.diff)
        });
      }

    },
    random_number(number){
      return Math.floor(Math.random() * Math.floor(number));
    },
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
  font-size: 7em;
  font-weight: bold;
  top: 35%;
  left: 50%;
  transform: translate(-50%, -50%);
}

.left_text {
  position: absolute;
  font-size: 6em;
  font-weight: bold;
  top: 35%;
  left: 14%;
  transform: translate(-50%, -50%);
}

.right_text {
  position: absolute;
  font-size: 6em;
  font-weight: bold;
  top: 35%;
  left: 86.5%;
  transform: translate(-50%, -50%);
}

</style>

