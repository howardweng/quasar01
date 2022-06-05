<template>
  <q-page class="q-pa-none flex justify-center">
    <q-card flat bordered class="my-card" style="width: 100%;">
      <q-card-section>
        <div class="text-h6">
          警示溫度
          <q-btn outline color="primary" :label="temperature.minTemp" />
          <q-btn flat color="primary" label=" <==> " />
          <q-btn outline color="primary" :label="temperature.maxTemp" />
          <q-btn flat icon="settings" to="/upload" label="" />
        </div>
      </q-card-section>

      <q-card-section class="q-pt-none">
        <!-- <div>
          <span class="cursor-pointer">{{ temperature.max }}======</span>
          <span class="cursor-pointer">{{ temperature.min }}</span>
        </div> -->
      </q-card-section>

      <q-separator inset />

      <q-card-section>
        <div class="scrollmenu" style="width: 100%;">
          <q-list>
            <q-item
              v-for="new_detect in new_detects"
              :key="new_detect.id"
              class="thumbnail_1"
            >
              <q-item-section>
                <img
                  :src="`img/${new_detect.photo}`"
                  alt=""
                  style="max-width: 200px; height: 150px;"
                />
              </q-item-section>

              <div class="col-12 text-center">
                <q-item-section>
                  <!-- <q-item-label color="red">{{detect.temperature}}</q-item-label> -->
                  <!-- <q-badge :label="detect.temperature" /> -->

                  <q-item-label
                    lines="1"
                    class="q-mt-xs text-body2 text-weight-bold text-negative "
                  >
                    <span class="cursor-pointer">{{
                      new_detect.temperature
                    }}</span>
                  </q-item-label>
                  <!-- <h4 style="font-size:2vw">{{detect.temperature}}</h4> -->
                  <q-item-label caption>{{ new_detect.time }}</q-item-label>
                </q-item-section>
              </div>
            </q-item>
          </q-list>
        </div>
      </q-card-section>
      <q-separator inset />

      <q-card-section>
        <div class="scrollmenu" style="width: 100%; ">
          <q-list>
            <q-item
              v-for="detect in detects"
              :key="detect.id"
              class="thumbnail"
            >
              <!-- <q-item-section top avatar>
                    <q-avatar rounded>
                      <img :src="`img/${detect.photo}`" alt=""
                      style="max-width: 300px; height: 150px;"
                      >
                    </q-avatar>
                  </q-item-section> -->
              <q-item-section>
                <img
                  :src="`img/${detect.photo}`"
                  alt=""
                  style="max-width: 100px; height: 75px;"
                />
              </q-item-section>

              <div class="col-12 text-center">
                <q-item-section>
                  <!-- <q-item-label color="red">{{detect.temperature}}</q-item-label> -->
                  <!-- <q-badge :label="detect.temperature" /> -->
                  <q-item-label
                    lines="1"
                    class="q-mt-xs text-body2 text-weight-bold text-negative "
                  >
                    <span class="cursor-pointer">{{ detect.temperature }}</span>
                  </q-item-label>
                  <!-- <h4 style="font-size:2vw">{{detect.temperature}}</h4> -->
                  <q-item-label caption>{{ detect.time }}</q-item-label>
                </q-item-section>
              </div>
            </q-item>
          </q-list>
        </div>
      </q-card-section>
    </q-card>

    <!-- <img src="data:image/png;base64,iVBORw0KGgoAAA
ANSUhEUgAAAAUAAAAFCAYAAACNbyblAAAAHElEQVQI12P4
//8/w38GIAXDIBKE0DHxgljNBAAO9TXL0Y4OHwAAAABJRU
5ErkJggg==" alt="Red dot" /> -->
    <br />
    <!-- <q-uploader
        url="http://192.168.1.100:1880/upload"
        style="max-width: 300px"
        :auto-upload="true"
      /> -->

    <!-- <br><br>
  <div class="col-4">
    <q-img
      :src= dataobj
      :ratio="16/9"
      style="max-width: 300px; height: 150px;"
    />
  </div> -->
    <!-- <div class="col-4">
    <q-img
      :src= dataobj
      :ratio="16/9"
      style="max-width: 300px; height: 150px;"
    />
  </div> -->
  </q-page>
</template>

<script>
import moment from "moment";
import axios from "axios";

export default {
  name: "PageIndex",
  data() {
    return {
      temperature: {},
      dataobj: "",
      new_detects: [],
      time: "",
      detects: [],
      new_detects: []
      //   detects: [
      //   {
      //     id: '1',
      //     temperature: '39.8°c',
      //     timestamp: '2022/04/30 13:30',
      //     photo: '100.jpg'
      //   },
      //   {
      //     id: '2',
      //     temperature: '38.9°c',
      //     timestamp: '2022/04/30 15:30',
      //     photo: '101.jpg'
      //   },
      //   {
      //     id: '3',
      //     temperature: '40.1°c',
      //     timestamp: '2022/04/30 13:30',
      //     photo: '102.jpg'
      //   }
      // ]
    };
  },
  created() {
    this.topicmqtt = "handwash/#"; // + this.MAC
    this.$mqtt.subscribe(this.topicmqtt);
    console.log("subscribe topic: ", this.topicmqtt);
    console.log("get axios: ");
    // axios
    //   .get("http://localhost:8080/temperature.txt")
    //   .then(res => {
    //     this.temperature = res.data;
    //     console.log("res.data: ", res.data);
    //   })
    //   .catch(error => {
    //     console.error("axios.post() error: ", error);
    //   });
  },
  mqtt: {
    "handwash/#"(data, topic) {
      // console.log('topic: ', topic)
      // this.dataobj = JSON.parse(data)
      // console.log("this.dataobj: ", this.dataobj)

      var last = this.detects;

      if (topic === "handwash/command/detect") {
        // this.dataobj = JSON.parse(data).img_base64]

        //this.new_detects = JSON.parse(data)
        var newtemp = JSON.parse(data);
        console.log("newtemp: ", newtemp);

        this.time = moment(new Date()).format("HH:mm:ss");

        // newtemp = newtemp.reverse()

        for (var i = 0; i < newtemp.length; i++) {
          newtemp[i].time = moment(new Date()).format("HH:mm:ss");
          delete newtemp[i].id;
          newtemp[i].id = i + 1;
        }
        //newtemp.reverse()
        // this.new_detects = newtemp
        // ********** process new array first before reverse

        Array.prototype.push.apply(last, newtemp);

        // ********* top new detect

        this.new_detects = newtemp.reverse();

        console.log("new_detects: ", this.new_detects);
        // this.new_detects = this.new_detects.reverse()

        // console.log("last: ", last)

        for (var i = 0; i < last.length; i++) {
          delete last[i].id;
          last[i].id = i + 1;
        }

        // last = (last.slice((last.length - 20), last.length))
        console.log("last: ", last);
        // last = (last.slice((last.length - 20), last.length))

        var temp = [];
        var DeviceList1 = last.map(function(a) {
          temp.push(a);
        });

        // this.detects = last
        //  console.log(this.detects.length)
        //  console.log("temp: ", temp)
        if (temp.length > 60) {
          temp = temp.slice(temp.length - 60, temp.length);
        }
        console.log("temp: ", temp);
        last = temp;
        this.detects = temp.reverse();
        // this.detects = this.detects.slice(0, 20);
        // this.detects = (this.detects.slice((this.detects.length - 20), this.detects.length))
        //this.detects = this.detects.push(this.new_detects)

        //  this.detects = this.detects.slice(0, 20);
        //  console.log(this.detects.length)
        //  console.log(this.detects)
        // this.device_data.mac = this.dataobj.mac
        // this.device_data.volt = this.dataobj.volt
        // this.device_data.gas = this.dataobj.gas
        // this.device_data.ir = this.dataobj.ir
        // this.device_data.image = 'data:image/jpeg;base64,'+ this.dataobj.img_bs64
        // this.device_data.time = moment(new Date()).format("HH:mm:ss"),

        // this.heatmap_data = {
        //   x_array: [],
        //   y_array: [],
        //   data: [],
        // }

        // this.above_customize_temperature_number = 0

        // for(var i = 0; i < 24; i++){
        //   for(var j = 0; j < 32; j++){
        //     this.heatmap_data.data.push([i, j, Math.round(this.device_data.ir[32 * i + j]*100)/100])

        //     if(this.device_data.ir[32 * i + j] >= 50){
        //       this.above_customize_temperature_number += 1
        //     }
        //   }
        //   this.heatmap_data.x_array.push(i+1)
        // }

        // for(var j = 0; j < 32; j++){
        //   this.heatmap_data.y_array.push(j+1)
        // }
      }
    }
  },
  mounted() {},
  methods: {},
  watch: {}
};
</script>

<style scoped>
div.scrollmenu {
  overflow: auto;
  white-space: wrap;
}

div.scrollmenu .thumbnail {
  display: inline-block;
  color: white;
  padding: 10px;
  width: 100px;
}
div.scrollmenu_1 {
  overflow: auto;
  white-space: wrap;
}

div.scrollmenu .thumbnail_1 {
  display: inline-block;
  color: white;
  padding: 10px;
  width: 180px;
}
</style>
