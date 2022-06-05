<template>
  <q-page class="flex flex-center">
    <div class="column items-center">
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

      <marquee
        width="1180px"
        height="100"
        class="text-h1"
        :scrollamount="marquee_speed"
        style="background-color:DodgerBlue;color:whitesmoke;"
        >{{ marquee_text }}</marquee
      >

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
      </div>

      <!-- <div class="column items-center">
        <div style="width: 800px; ">
          <q-input
            class="zip-input"
            filled
            v-model="dataobj.customer_talk"
            dense
            readonly
          >
            <template v-slot:prepend>
              <q-icon name="record_voice_over" />
            </template>
          </q-input>
        </div>
        <br />

        <div style="width: 800px">
          <q-input
            class="zip-input"
            filled
            v-model="dataobj.items"
            dense
            readonly
          >
            <template v-slot:prepend>
              <q-icon name="shopping_cart" />
            </template>
          </q-input>
        </div>
        <br />
        <div style="width: 800px">
          <q-input
            type="textarea"
            filled
            v-model="dataobj.store_talk"
            dense
            readonly
          >
            <template v-slot:prepend>
              <q-icon name="reply" />
            </template>
          </q-input>
        </div>
        <br />
        <div style="width: 800px">
          <q-input
            type="textarea"
            filled
            v-model="dataobj.location_talk"
            dense
            readonly
          >
            <template v-slot:prepend>
              <q-icon name="place" />
            </template>
          </q-input>
        </div>
        <br />

        <div style="width: 800px">
          <q-input
            class="zip-input"
            filled
            v-model="dataobj.time"
            dense
            readonly
          >
            <template v-slot:prepend>
              <q-icon name="timer" />
            </template>
          </q-input>
        </div>
      </div> -->

      <div class="column items-center q-pt-md" v-show="false">
        <q-media-player
          type="audio"
          background-color="black"
          :muted="false"
          :autoplay="false"
          :sources="store_request"
          :loop="false"
          :disabled-seek="false"
          :hide-volume-btn="false"
          :no-controls="false"
          ref="qmp"
          @ended="onEnded"
        ></q-media-player>
      </div>

      <div class="column items-center q-pt-md" v-show="false">
        <q-media-player
          type="audio"
          background-color="black"
          :muted="false"
          :autoplay="true"
          :sources="store_request1"
          :loop="false"
          :disabled-seek="false"
          :hide-volume-btn="false"
          :no-controls="false"
          ref="qmp1"
          @ended="ended_few_tries"
        ></q-media-player>
      </div>

      <div class="row">
        <q-btn
          size="xl"
          class="q-ml-sm"
          icon="play_arrow"
          @click="$refs.qmp.togglePlay()"
          outline
        ></q-btn>
      </div>

      <q-page-sticky
        v-if="btnStop"
        position="bottom-right"
        :offset="[720, 520]"
        style="z-index: 10000"
      >
        <q-btn round size="30px" icon="stop" color="negative" @click="stop()" />
      </q-page-sticky>

      <q-page-sticky v-if="btnAsk" position="top" :offset="[1070, 250]">
        <div style="width: 800px; font-size:40px">
          倒數 <span id="time">{{ seconds }}</span> 秒
        </div>
      </q-page-sticky>
    </div>
  </q-page>
</template>

<script>
import { QSpinnerBars } from "quasar";
import moment from "moment";
import axios from "axios";

export default {
  name: "PageIndex",
  data() {
    return {
      detected: false,
      marquee_speed: "",
      marquee_text: "",
      TempLabel: {
        min: "",
        max: ""
      },
      // marquee_text:"目前防疫溫度偵測進行中，如體溫過高者將發出提醒畫面與聲音，請配合警衛再進一步量測溫度，如造成您的不便敬請見諒!!!***謝謝***",
      intervalId: null, //data 定義一個定時器id
      top_video: [
        {
          // src: 'https://ftp.nluug.nl/pub/graphics/blender/demo/movies/ToS/ToS-4k-1920.mov',
          src: "/top.mov",
          type: "video/mp4"
        }
      ],
      center_video: [
        {
          // src: 'https://ftp.nluug.nl/pub/graphics/blender/demo/movies/ToS/ToS-4k-1920.mov',
          src: "/center.mov",
          type: "video/mp4"
        }
      ],
      temperature: 0,

      nothing_count: 0,
      sentStatus: false,
      intervalId: 0,
      minutes: "05",
      // seconds: "60",
      seconds: "05",
      text: "test",
      dataobj: {},
      mediaRecorder: null,
      chunks: [],
      audios: [],
      btnStop: false,
      btnAsk: false,
      temp_audio: {},
      store_request1: [],
      store_request: [
        {
          // src: 'https://ftp.nluug.nl/pub/graphics/blender/demo/movies/ToS/ToS-4k-1920.mov',
          src: "http://192.168.1.100:8080/store/store_request.mp3",
          type: "audio/mp3"
        }
      ],
      store_reply_items: []
    };
  },
  created() {
    // Subscribe mqtt
    this.subscribe_topic = "nlp/#";
    this.$mqtt.subscribe(this.subscribe_topic);
    console.log("subscribe(nlp/#)");
    // var hide = this.hideControls()
    // console.log(hide)
    let one = "http://localhost:8080/marquee.txt"
    let two = "http://localhost:8080/temperature.txt"

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
  mqtt: {
    "nlp/#"(data, topic) {
      console.log("topic: ", topic);
      this.dataobj = JSON.parse(data);
      this.dataobj.time = moment(new Date()).format("HH:mm:ss");
      // console.log("this.dataobj: ", this.dataobj)
      // var last = this.detects;
      if (topic === "nlp/from_store/store_reply_nothing") {
        // this.dataobj = JSON.parse(data).img_base64]
        console.log("this.dataobj: ", this.dataobj);

        if (this.nothing_count == 0) {
          this.nothing_count = 1;
        } else {
          this.nothing_count = this.nothing_count + 1;
        }

        console.log(this.nothing_count);

        if (this.nothing_count < 4) {
          // this.talkaudio = new Audio(this.dataobj.reply_url);
          // console.log("Play OK sound");
          // const talkaudioPromise = this.talkaudio.play();

          this.store_request1 = [
            {
              // src: this.dataobj.reply_url,
              src: "http://192.168.1.100:8080/store/store_request.mp3",
              type: "audio/mp3"
            }
          ];

          console.log("this.$refs.qmp1.play()");
          this.$refs.qmp1.play();
        } else {
          console.log(
            "=======4th tried, wait for 30 seconds to try again =========="
          );
        }
      }
      if (topic === "nlp/from_customer/customer_ask_items") {
        // this.dataobj = JSON.parse(data).img_base64]
        console.log("this.dataobj: ", this.dataobj);
        //this.new_detects = JSON.parse(data)
      }
      if (topic === "nlp/from_store/store_reply_items") {
        // this.dataobj = JSON.parse(data).img_base64]
        console.log("this.dataobj: ", this.dataobj);

        // audio play()

        // this.store_request = [
        //   {
        //     // src: 'https://ftp.nluug.nl/pub/graphics/blender/demo/movies/ToS/ToS-4k-1920.mov',
        //     src: this.dataobj.reply_url,
        //     type: "audio/mp3"
        //   }
        // ];

        this.talkaudio = new Audio(this.dataobj.reply_url);
        console.log("Play OK sound");
        const talkaudioPromise = this.talkaudio.play();
        console.log(talkaudioPromise);
        // if (talkaudioPromise !== undefined) {
        //   talkaudioPromise
        //     .then(() => {
        //       console.log("Finished Play talkaudio sound");
        //       // Automatic playback started!
        //       // Show playing UI.
        //     })
        //     .catch(error => {
        //       // Auto-play was prevented
        //       // Show paused UI.
        //     });
        // }

        //this.new_detects = JSON.parse(data)
      }
    }
  },
  mounted() {
    // this.startTimer(60 * 5)
    // this.startTimer(10);
    this.init();
  },
  beforeDestroy() {
    clearInterval(this.intervalId);
  },
  methods: {
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
    },
    init() {
      if (navigator.mediaDevices.getUserMedia) {
        navigator.mediaDevices
          .getUserMedia({ audio: true })
          .then(stream => {
            this.mediaRecorder = new MediaRecorder(stream);
            this.mediaRecorder.ondataavailable = e => {
              // console.log(e.data)

              this.chunks.push(e.data);
              //  console.log(e.data)
            };

            this.mediaRecorder.onstop = e => {
              // const blob = new Blob(this.chunks, { type: 'audio/wav; codecs="1"' })
              const blob = new Blob(this.chunks, {
                type: 'audio/mpeg; codecs="mp3"'
              });
              // const blob = new Blob(this.chunks, { type: 'audio/ogg; codecs=vorbis' })

              console.log(blob);

              const myFile = new File([blob], "ask.mp3", {
                // const myFile = new File([blob], 'test_test.wav', {
                type: blob.type
              });
              console.log(myFile);
              // console.log(myFile instanceof File);
              // console.log(myFile instanceof Blob);

              const formData = new FormData();
              // formData.append('test_test.wav', blob);
              fetch("http://192.168.1.100:1880/upload_speech", {
                method: "POST",
                body: myFile
              });

              // const formData = new FormData();
              // formData.append('test_test.wav', blob);
              // fetch('http://192.168.1.100:1880/upload_speech', {
              //   method: 'POST',
              //   body: formData
              // });

              const audioURL = window.URL.createObjectURL(blob);
              // console.log(audioURL)

              this.temp_audio = blob;

              // console.log(audioURL)
              this.chunks = [];
              this.audios.push(audioURL);

              // console.log(this.audios)
              // this.audios = audioURL
            };
          })
          .catch(function(err) {
            console.log("The following getUserMedia error occured: " + err);
          });
      } else {
        alert("getUserMedia not supported on your browser!");
      }
    },
    onEnded() {
      // this.$q.loading.show({
      //   spinner: QSpinnerBars,
      //   spinnerColor: "white",
      //   backgroundColor: "primary"
      // });
      console.log("asking ended, begin to ask");
      this.startTimer(5);
      console.log("asking audio begin, recording for 5 seconds, this.record()");
      this.btnAsk = true;
      this.record();
    },
    startTimer(duration, display) {
      var timer = duration,
        minutes,
        seconds;
      this.intervalId = setInterval(() => {
        this.minutes = parseInt(timer / 60, 10);
        this.seconds = parseInt(timer % 60, 10);

        this.minutes = this.minutes < 10 ? "0" + this.minutes : this.minutes;
        this.seconds = this.seconds < 10 ? "0" + this.seconds : this.seconds;

        if (--timer < 0) {
          timer = duration;
          console.log("end counting, this.stop()");
          this.stop();
          // this.end_ask();
          clearInterval(this.intervalId);
        }
      }, 1000);
    },
    end_ask() {
      this.btnAsk = false;
      this.$q.loading.hide();

      // console.log("sending audio for process, Destroy only do once");

      // console.log("this.sentStatus");
      //  console.log(this.sentStatus);

      // if (this.sentStatus === false) {

      //   console.log("sending once for process")
      //   this.sentStatus = true
      //   // this.beforeDestroy()

      // } else {
      //   console.log("already sent, ignore")
      // }
      // this.beforeDestroy()
      //this.intervalId();
    },
    ask() {
      this.btnAsk = false;
      this.$q.loading.hide();
    },
    record() {
      this.mediaRecorder.start();
      this.$q.loading.show({
        spinner: QSpinnerBars,
        spinnerColor: "white",
        backgroundColor: "primary"
      });
      // this.btnStop = true;
    },
    stop() {
      this.mediaRecorder.stop();
      this.$q.loading.hide();
      // this.btnStop = false;
      this.btnAsk = false;
      this.$q.loading.hide();
    },
    ended_few_tries() {
      console.log("ended_few_tries, then do something");
    }
  }
};
</script>

<style>
.zip-input {
  height: 50px;
  font-size: 25px;
}
</style>
