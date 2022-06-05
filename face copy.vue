<template>
  <q-card>
    <q-card-section>


      <div class="text-center"  id="video">
        <iframe src="http://localhost:1880/simple" allow="camera *;microphone *" width="840" height="840"
          style="border:none;" allow-same-origin></iframe>
            <div class="text-center"  id="startbutton" :key="componentKey">
            {{item}}<br>{{percent}}-{{item_count}}
            </div>

      </div>




    </q-card-section>

        <q-card-section>
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
    </q-card-section>
    <q-card-section>
      <q-btn @click="capture" label="Capture" />
    </q-card-section>


  </q-card>

  
</template>

<script>
import adapter from 'webrtc-adapter'

// let componentKey = 0

export default {
  data() {
    return {
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
    console.log("this.$mqtt.subscribe(this.subscribe_topic)", this.subscribe_topic)

  },
  // mounted() {


  //   window.addEventListener('message', receiveMessage, false);
  //   function receiveMessage(event) {
  //               // alert("got message: " + event.data);
  //         console.log(event.data)
  //         console.log(typeof(event.data))
  //         this.receive = event.data
  //     }

  // },
  async mounted() {
    await this.startWebcam()

    let _this = this;

    window.addEventListener('message', receiveMessage, false);
    function receiveMessage(event) {
                // alert("got message: " + event.data);
          // console.log(event.data)
          // console.log(typeof(event.data))
          _this.receive = event.data
          // console.log(_this.receive)



          // var start = new Date().getTime();
          // var i
          // for (i = 0; i < 50000; ++i) {
          // // do something
          //  console.log("something")

          // }

          // var end = new Date().getTime();
          // var time = end - start;
          // console.log('Execution time: ' + time);



      }

  },
  async destroyed() {
    this.stopWebcam()
  },
  beforeDestroy() {  
    this.cancelAutoUpdate();  
  },  
  computed: {

  },
  methods: {

    async fetchData() {  
      // const res = await fetch("https://yesno.wtf/api");  
      // const data = await res.json();  
      // this.answer = data;  

      // console.log("regular----update")
      // console.log(this.item)
      

      if(this.item === this.last_item) {

      //  console.log("item_count + 1")
       this.item_count  = this.item_count + 1


            if(this.item_count == 2 && this.item !== "empty") {

              // console.log("posting data")

                const url = "http://localhost:1880/post_in";
                fetch(url, {
                    method : "POST",
                    body: this.item,

                }).then(


                    response => console.log(response.text()) ,
                    // console.log(response.text())
                    // same as function(response) {return response.text();}

                    this.capture() 


                ).then(
                    html => console.log(html)
                );

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
      clearInterval(this.timer);  
    }, 

    // update_item: function() {

    //    setInterval(() => {
    //       this.myText = componentKey + 1
    //     }, 1000);
    //  }
    //   console.log("update_time here.....")
    // },


    // update_item: function() {

    //  this.componentKey += 1;  
    //  console.log("update_time here.....")

    // },
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

          var filename = timestamp +"_capture.png"       


          const formData = new FormData();
          // formData.append('test_test.wav', blob);
          fetch("http://localhost:1880/upload_img?name=" + this.receive.split('/')[0] + "&filename=" + filename, {
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

            console.log('filename: =======>',filename)

            var file_url = "http://localhost:8080/face/" + filename

            console.log('file_url : =======>',file_url )
            

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
