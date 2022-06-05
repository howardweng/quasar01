<template>
  <q-page class="q-pa-none flex justify-center">

  <q-card>
    <q-card-section>

      <div class="text-center"  id="video">
        <iframe src="http://localhost:1880/simple" allow="camera *;microphone *" width="840" height="840"
          style="border:none;" allow-same-origin></iframe>
            <div class="text-center"  id="startbutton" :key="componentKey">
            {{item}}<br>
            <div style="font-size: 10px;"> 
            {{percent}} - {{item_count}}
            </div>
            </div>

            <div id="startbutton2">
            <q-img
              src="http://localhost:8080/face1.png"
              :ratio="1"
            />
            </div> 

            

            <div class="col-4" id="startbutton1"  v-show="show_detected">            
            <q-img
              :src="detected.file_url"
              :ratio="1"
              style="max-width: 150px;  transform: rotateY(180deg);"
            />
            <br>{{detected.time}} - 36.7°C
          </div>

      </div>

      <!-- <q-card-section>
          <div class="col-4" id="video">            
            <q-img
              :src="detected.file_url"
              :ratio="1"
              style="max-width: 100px; height: 75px; transform: rotateY(180deg);"
            />
            <br>{{detected.time}} 
          </div>
    </q-card-section> -->

    </q-card-section>


    <!-- <q-card-section>
      <q-btn @click="capture" label="Capture" />
    </q-card-section> -->



  </q-card>

        <div class="text-center"  id="video" v-show="show">
        <video
          ref="webcam"
          autoplay
          playsinline
          muted
          width="400"
          height="400"
        />
      </div>

</q-page>  
</template>

<script>
// import adapter from 'webrtc-adapter'
import moment from "moment";

// let componentKey = 0

export default {
  data() {
    return {
      show_detected:false,
      intervalId: 0,
      minutes: '00',
      seconds: '60',
      detected:"",
      file_url:"", 
      time:"",
      subscribe_topic:'',
      show:false,
      mediaStream: null,
      imageCapture: null,
      captured: null,
      receive: "initial",
      value:89,
      display: "",
      item : "",
      percent: 0,
      componentKey:0,
      last_item:"",
      item_count:0 
      
    }
  },
  watch:{

    receive:{
      handler: function() {
        // console.log("&&&&&&&&&&&&here is what happen", this.receive)
        this.display = this.receive 
        this.item = this.receive.split('/')[0]
        this.percent = this.receive.split('/')[1] *100 + "%"    
        },
    },
    // componentKey:{      
    //   handler: function() {
    //     console.log("*********this.display", this.display)
    //   }
    // }

  },
  created() {


    this.fetchData();  
    this.timer = setInterval(this.fetchData, 500); 

    this.subscribe_topic = 'face/#'
    this.$mqtt.subscribe(this.subscribe_topic)
    console.log("subscribe_topic", this.subscribe_topic)

  },
  mqtt: {
    "face/#"(data, topic) {
      console.log('topic: ', topic)
      this.dataobj = JSON.parse(data)
      // console.log("this.dataobj: ", this.dataobj)

      var last = this.detects;

      if (topic === "face/from_server/detected") {

        this.detected = JSON.parse(data);

        console.log("newtemp: ", this.detected);

        this.show_detected = true

        // this.time = moment(new Date()).format("HH:mm:ss");


      }
    }
  },
  async mounted() {
    await this.startWebcam()

    let _this = this;

    window.addEventListener('message', receiveMessage, false);
    function receiveMessage(event) {
          _this.receive = event.data
    }

  },
  async destroyed() {
    this.stopWebcam()
  },
  beforeDestroy() {  
    this.cancelAutoUpdate();  
    clearInterval(this.intervalId)
  },  
  computed: {

  },
  methods: {
    startTimer (duration) {
      var timer = duration

      this.intervalId = setInterval(() => {
        if (--timer < 0) {
          timer = duration

          console.log("*******image gone************")

          this.cancelAutoUpdate()

          // clearInterval(this.intervalId)
          // //當清除定時器之後，重新讓intervalId為null
          // this.intervalId = null;

          this.show_detected = false
        }
      }, 5000)
    },
    async fetchData() {  

      if(this.item === this.last_item) {


       this.item_count  = this.item_count + 1


            if(this.item_count == 2 && this.item !== "Empty" && this.item !== "Unknown") {
             this.capture()        

            }


      } else {
        console.log("item_count ==> 0")
       this.item_count  = 0 
      }
      
      // console.log("set last_item")
      this.last_item = this.item

      // console.log("item_count")
      // console.log(this.item_count)
     

    },  
    cancelAutoUpdate() {  
      // clearInterval(this.timer);  
      clearInterval(this.intervalId)
      //當清除定時器之後，重新讓intervalId為null
      this.intervalId = null;
    }, 

    async startWebcam() {
      var constraints = {
        audio: false,
        video: {
          facingMode: 'environment',
          width: 400,
          height: 400
        }
      }
      try {
        this.mediaStream = await navigator.mediaDevices.getUserMedia(
          constraints
        )
        var video = this.$refs.webcam
        video.srcObject = this.mediaStream
        video.onloadedmetadata = function (e) {
          video.play()
        }
        const track = this.mediaStream.getVideoTracks()[0]
        this.imageCapture = new ImageCapture(track)

        // this.item_count  = 0

      } catch (err) {
        console.log(err.name + ': ' + err.message)
      }
    },
    async capture() {
      const vm = this
      try {
        const blob = await this.imageCapture.takePhoto().then(blob => {
          console.log('Took photo:', blob)
          vm.$emit('capture', blob)


          const myFile = new File([blob], "capture.png", {
            // const myFile = new File([blob], 'test_test.wav', {
            type: blob.type
          });
          console.log(myFile);
          // console.log(myFile instanceof File);
          // console.log(myFile instanceof Blob);

          var timestamp = Date.now()
          var filename = this.receive.split('/')[0]+ '_' + timestamp +"_capture.png"       

          var time = moment(new Date()).format("HH:mm:ss");

          var file_url = "http://localhost:8080/face/" + this.receive.split('/')[0] + '/'+ filename 

          var filepath = "/home/datavan/quasar/public/face/" + this.receive.split('/')[0] + '/'+  filename

          // console.log(this.item)


          const formData = new FormData();
          // formData.append('test_test.wav', blob);
          fetch("http://localhost:1880/upload_img?name=" 
          + this.receive.split('/')[0] 
          + "&filename=" + filename+ "&time=" + time 
          + "&file_url=" + file_url+ "&timestamp=" 
          + timestamp+ "&filepath=" + filepath
          + "&timestring=" + moment(new Date()).format("YYYY-MM-DD HH:mm:ss")
          , 
          {
            method: "POST",
            body: myFile,
            // item : this.receive.split('/')[0]
          })
          .then(response => response)
          .then(data => {

            console.log('Success:', data.url);
            console.log('data:', data);
            const url = new URL(data.url);

            var name = url.searchParams.get('name')

            console.log('name: =======>',name)

            const filename = url.searchParams.get('filename')

            //console.log('filename: =======>',filename)

            var file_url = url.searchParams.get('filepath')




            console.log('file_url : =======>', file_url )

            // this.show_time()

            this.startTimer(0)


         

          })
          .catch((error) => {
            console.error('Error:', error);
          });

        })
      } catch (error) {
        console.log('takePhoto() error: ', error)
      }
    },
    async stopWebcam() {
      this.mediaStream.getTracks().forEach(track => {
        track.stop()
      })
    }
  }
}
</script>

<style lang="scss" scoped>
// video {
//   width: 100%;
//   height: 100%;
//   // border: 1px solid blue;
// }

// img {
//     /* flip horizontally */
//     transform: rotateY(180deg);
//   // border: 1px solid blue;
// }

video {
  width: 550px;
  height: 550px;
  -webkit-transform: scaleX(-1);
  transform: scaleX(-1);
  // border: 1px solid blue;
}

#startbutton {
  width: 150px;
  height: 70px;
  display: block;
  position: relative;
  margin-left: auto;
  margin-right: auto;
  bottom: 120px;
  background-color: rgba(0, 150, 0, 0.5);
  border: 1px solid rgba(255, 255, 255, 0.7);
  box-shadow: 0px 0px 1px 2px rgba(0, 0, 0, 0.2);
  font-size: 24px;
  font-family: "Lucida Grande", "Arial", sans-serif;
  color: rgba(255, 255, 255, 1.0);
}


</style>
<style lang="scss" scoped>
video1 {
  width: 75vmin;
  height: 75vmin;
  -webkit-transform: scaleX(-1);
  transform: scaleX(-1);
  // border: 1px solid blue;
}

#startbutton1 {
  display: block;
  position: relative;
  margin-left: 620px;
  margin-right: auto;
  bottom: 790px;
  width: 150px;
  background-color: rgba(255, 255, 255, 1.0);
}


#startbutton2 {
  display: block;
  position: relative;
  margin-right: auto;
  margin-left: 240px;
  bottom: 700px;
  width: 320px;
}

</style>
