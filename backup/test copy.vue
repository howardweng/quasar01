<template>
  <q-page class="q-pa-lg">
    <div class="column items-center">
 <div  style="width: 1000px; font-size:50px">
  <!-- <div>count in <span id="time">{{ minutes }}:{{ seconds }}</span> seconds!</div> -->
    <div>count in <span id="time">{{ seconds }}</span> seconds!</div>
  </div>
</div>
  </q-page>
</template>

<script>
import { QSpinnerBars } from "quasar";
import moment from "moment";

export default {
  name: "PageIndex",
  data() {
    return {
      intervalId: 0,
      minutes: '00',
      seconds: '60'
    };
  },
  created() {
    // Subscribe mqtt
    this.subscribe_topic = "nlp/#";
    this.$mqtt.subscribe(this.subscribe_topic);
    console.log("subscribe(nlp/#)");
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
      ]

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
  mounted () {
    // this.startTimer(60 * 5)
    this.startTimer(10)
  },
  beforeDestroy () {
    clearInterval(this.intervalId)
  },
  methods: {
    startTimer (duration, display) {
      var timer = duration,
        minutes,
        seconds
      this.intervalId = setInterval(() => {
        this.minutes = parseInt(timer / 60, 10)
        this.seconds = parseInt(timer % 60, 10)

        this.minutes = this.minutes < 10 ? "0" + this.minutes : this.minutes
        this.seconds = this.seconds < 10 ? "0" + this.seconds : this.seconds

        if (--timer < 0) {
          timer = duration

          console.log("done, popup")
        }
      }, 1000)
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
