<template>
  <q-page class="q-pa-md">
    <div class="column items-center">
      <div class="col-3 q-gutter-xl">
        <q-btn color="primary" push size="70px" icon="mic" @click="record()">
          ask
        </q-btn>
        <!-- <q-btn color="primary" push size="lg" icon="stop" @click="stop()">Stop</q-btn> -->
      </div>

      <br />

      <div style="width: 800px; font-size:30px">
        發問產品位置，如：請問 麵粉，肥皂 在哪裡？
        <div style="width: 800px; font-size:20px">
          (目前 麵粉 肥皂 水果 有位置)
        </div>
      </div>
      <br />

      <br />
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

      <!-- <div style="width: 800px">
        <q-input
          class="zip-input"
          filled
          v-model="dataobj.reply_url"
          dense
          readonly
        >
          <template v-slot:prepend>
            <q-icon name="folder" />
          </template>
        </q-input>
      </div>
      
      <br /> -->
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
        <q-input class="zip-input" filled v-model="dataobj.time" dense readonly>
          <template v-slot:prepend>
            <q-icon name="timer" />
          </template>
        </q-input>
      </div>
    </div>

    <div class="column items-center q-pt-md">
      <q-media-player
        type="audio"
        background-color="black"
        :muted="false"
        :autoplay="false"
        :sources="top_video"
        :loop="false"
        :disabled-seek="false"
        :hide-volume-btn="false"
        :no-controls="false"
      ></q-media-player>
    </div>

    <q-page-sticky
      v-if="btnStop"
      position="bottom-right"
      :offset="[720, 520]"
      style="z-index: 10000"
    >
      <q-btn round size="30px" icon="stop" color="negative" @click="stop()" />
    </q-page-sticky>
  </q-page>
</template>

<script>
import { QSpinnerBars } from "quasar";
import moment from "moment";

export default {
  name: "PageIndex",
  data() {
    return {
      text: "test",
      dataobj: {},
      mediaRecorder: null,
      chunks: [],
      audios: [],
      btnStop: false,
      temp_audio: {},
      top_video: []
      // top_video: [
      //   {
      //     // src: 'https://ftp.nluug.nl/pub/graphics/blender/demo/movies/ToS/ToS-4k-1920.mov',
      //     src: "http://localhost:8080/ask.mp3",
      //     type: "audio/mp3"
      //   }
      // ]
    };
  },
  created() {
    // Subscribe mqtt
    this.subscribe_topic = "nlp/#";
    this.$mqtt.subscribe(this.subscribe_topic);
    console.log("subscribe(nlp/#)");
    // var hide = this.hideControls()
    // console.log(hide)
  },
  mqtt: {
    "nlp/#"(data, topic) {
      console.log("topic: ", topic);
      this.dataobj = JSON.parse(data);
      this.dataobj.time = moment(new Date()).format("HH:mm:ss");
      // console.log("this.dataobj: ", this.dataobj)
      // var last = this.detects;
      if (topic === "nlp/from_customer/customer_ask_items") {
        // this.dataobj = JSON.parse(data).img_base64]
        console.log("this.dataobj: ", this.dataobj);
        //this.new_detects = JSON.parse(data)
      }
      if (topic === "nlp/from_store/store_reply_items") {
        // this.dataobj = JSON.parse(data).img_base64]
        console.log("this.dataobj: ", this.dataobj);

        // audio play()

        this.top_video = [
          {
            // src: 'https://ftp.nluug.nl/pub/graphics/blender/demo/movies/ToS/ToS-4k-1920.mov',
            src: this.dataobj.reply_url,
            type: "audio/mp3"
          }
        ];

        this.talkaudio = new Audio(this.dataobj.reply_url);
        console.log("Play OK sound");
        const talkaudioPromise = this.talkaudio.play();
        console.log(talkaudioPromise);
        if (talkaudioPromise !== undefined) {
          talkaudioPromise
            .then(() => {
              console.log("Finished Play talkaudio sound");
              // Automatic playback started!
              // Show playing UI.
            })
            .catch(error => {
              // Auto-play was prevented
              // Show paused UI.
            });
        }

        //this.new_detects = JSON.parse(data)
      }
    }
  },
  mounted() {
    this.init();
  },
  methods: {
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
              // fetch('http://192.168.1.188:1880/upload_speech', {
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
    record() {
      this.mediaRecorder.start();
      this.$q.loading.show({
        spinner: QSpinnerBars,
        spinnerColor: "white",
        backgroundColor: "primary"
      });
      this.btnStop = true;
    },
    stop() {
      this.mediaRecorder.stop();
      this.$q.loading.hide();
      this.btnStop = false;
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
