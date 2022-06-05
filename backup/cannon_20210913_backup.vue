<template>
  <div v-if="loading_finish" class="q-pa-md row justify-center">
    <q-card flat class="q-pa-md row justify-center" style="width: 80%">
      <div class="col-12">
        <img
          basic
          no-default-spinner
          class="col-12"
          style="width: 100%;"
          :src="picture"
        />
      </div>
      <div class="q-pa-md col-12 row justify-center" style="font-size:2.5rem;">
        <q-card class="q-pa-xs" flat>
          <span v-if="processing" class='col-12 row justify-center' style="font-weight:bold; color:blue;"> {{ list[original_position ]}} </span>
          <span v-if="!processing" class='col-12 row justify-center' style="font-weight:bold; color:blue;"> {{ position_text }} </span>
        </q-card>
        <!-- <span class='q-pa-sm col-12 row justify-center' style="font-size:1rem; color:grey;">目標位置： {{list[target_position]}}</span> -->
      </div>
    </q-card>

    <!-- <div class="q-pa-sm row col-12 justify-center items-center">
      <q-input class="q-pa-sm col-md-2 col-sm-3" outlined v-model="position" label="位置" />/
      <q-input class="q-pa-sm col-md-2 col-sm-3" outlined v-model="direction" label="方向" />/
      <q-input class="q-pa-sm col-md-2 col-sm-3" outlined v-model="speed" label="速度" />
      <q-btn :disable="processing" class="q-pa-sm flex flex-center col-md-1 col-sm-2" style="height:75%" no-caps label="執行" @click="process(position + '/' + direction + '/' + speed)"/>
    </div> -->
  </div>
</template>

<script>

function switch_rule(list, number){
  switch(list[number]){
    case list[number]:
      return require("../images/" + list[number] + ".png")
  }
}

function switch_vague_rule(vague_list, number){
  switch(vague_list[number]){
    case vague_list[number]:
      return require("../images/vague_" + number + ".png")
  }
}

import {Loading} from 'quasar'

export default {
  name: 'cannon',
  data () {
    return {
      diff: 0,
      
      position: '',
      direction: '',
      speed: '',
      loading_finish: false,
      picture: null,
      original_position: 0,
      target_position: 0,
      position_text: 0,

      processing: false,
      list: [],
      vague_list: [1, 2, 3, 4, 5],

      loading_message: "<b>系統載入中...</b>",
    }
  },
  created () {
    
    // 【Initialization】Start
    Loading.show({
      message: this.loading_message,
      customClass: 'text-h5'
    })

    // Subscribe mqtt
    this.subscribe_topic = 'cannon/#'
    this.$mqtt.subscribe(this.subscribe_topic)

    this.created_publish = 'created'
    this.$mqtt.publish('quasar/created', this.created_publish)

    // 初始化圖片及資料
    for (var i = 0; i < 128; i++){
      this.list.push(i*50)
    }
    
    var result
    result = switch_rule(this.list, this.original_position)
    this.picture = result


  },
  mounted () {

  },
  mqtt: {
    'cannon/#' (data, topic) {
      this.object = JSON.parse(data)

      console.log("topic: ", topic, " this.object: ", this.object)

      if(topic === 'cannon/created'){
        // 【Initialization】End
        if(this.object.result === 'legal'){
          
          setTimeout(() => {
            Loading.hide()
            this.loading_finish = true
          }, 1000)
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
          this.process(this.object.command)
        }
      }

    }
  },
  methods: {
    process(process_text){
      this.processing = true

      let result
      let time_interval

      // this.process_text = this.position + '/' + this.direction + '/' + this.speed

      // console.log("this.process_text: ", this.process_text)

      this.process_position = process_text.split('/')[0]
      this.process_direction = process_text.split('/')[1].toLowerCase()
      this.process_speed = process_text.split('/')[2].toLowerCase()

      this.target_position = Math.floor(this.process_position / 50)

      this.position_text = this.process_position


      console.log("original_position: ", this.original_position, "\ntarget_position: ", this.target_position)

      if(this.target_position > 128 || this.target_position < 0 || this.process_position > 6400){
        console.log("Over range!")
        this.processing = false
        return
      }

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
      else{
        time_interval = 50
      }

        // 決定轉動方向
      if(this.process_direction === 'left'){

        if(this.target_position > this.original_position){
          // Ex. 原本在 3000 (60)，新位置設定 4000 (80)，距離 diff 為 128 - 20 = 108 ( 60 -> 0(128) -> 80)
          this.diff = 128 - Math.abs(this.target_position - this.original_position)
        }
        else{
          this.diff = Math.abs(this.target_position - this.original_position)
        }


        for (let i = 0; i < this.diff; i+=1){
          setTimeout(() => {
            this.original_position -= 1
            
            if(this.original_position < 0){
              this.original_position += 128
            }

            if(i !== this.diff - 1){
              result = switch_vague_rule(this.vague_list, (i%5 + 1))
            }
            else{
              result = switch_rule(this.list, this.original_position)
            }
            
            this.picture = result
            }, time_interval * (i+1)
          );
        }

        }
      else if (this.process_direction === 'right'){

        if(this.target_position < this.original_position){
            // Ex. 原本在 3000 (60)，新位置設定 2000 (40)，距離 diff 為 128 - 20 = 108 ( 60 -> 128(0) -> 40)
            this.diff = 128 - Math.abs(this.target_position - this.original_position)
        }
        else{
            this.diff = Math.abs(this.target_position - this.original_position)
        }

        console.log("this.diff: ", this.diff)

        let reverse_count = this.diff

        for (let i = 0; i < this.diff; i+=1){
          setTimeout(() => {
            this.original_position += 1

            if(this.original_position > 127){
                this.original_position -= 128
            }

            if(i !== this.diff - 1){
                result = switch_vague_rule(this.vague_list, (reverse_count%5 + 1))
            }
            else{
                result = switch_rule(this.list, this.original_position)
            }
            
            this.picture = result
            reverse_count -= 1
          }, time_interval * (i+1));
        }

      }
      else {
        this.processing = false
        return
      }

        setTimeout(() => {
          console.log("Finish. original_position: ", this.original_position, "\ntarget_position: ", this.target_position)
          console.log("-----------------")
          this.processing = false
        }, time_interval * this.diff)

    },
    random_number(number){
      return Math.floor(Math.random() * Math.floor(number));
    },
  }
}
</script>


<style lang="scss">

</style>

