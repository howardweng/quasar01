<template>
  <q-card>
    <q-card-section>
      <div class="text-center">
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
export default {
  data() {
    return {
      mediaStream: null,
      imageCapture: null,
      captured: null
    }
  },
  async mounted() {
    await this.startWebcam()
  },
  async destroyed() {
    this.stopWebcam()
  },
  methods: {
    async startWebcam() {
      var constraints = {
        audio: false,
        video: {
          facingMode: 'user',
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
        video.onloadedmetadata = function(e) {
          video.play()
        }
        const track = this.mediaStream.getVideoTracks()[0]
        this.imageCapture = new ImageCapture(track)
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
video {
  width: 75vmin;
  height: 75vmin;
  -webkit-transform: scaleX(-1);
  transform: scaleX(-1);
  // border: 1px solid blue;
}
</style>

