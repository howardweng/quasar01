<template>
  <div>
    <!-- <q-btn color="primary" label="Get Picture" @click="captureImage" /> -->

    <img :src="imageSrc">
  </div>
</template>

<script>
import { Camera, CameraResultType } from '@capacitor/camera';

export default {
  data () {
    return {
      imageSrc: ''
    }
  },
  created() {
    // Subscribe mqtt
    this.captureImage()
  },
  methods: {
    async captureImage () {
      const image = await Camera.getPhoto({
        quality: 90,
        allowEditing: true,
        resultType: CameraResultType.Uri,
        presentationStyle:'fullscreen',
        allowEditing:false,
        saveToGallery:false
      })
      // The result will vary on the value of the resultType option.
      // CameraResultType.Uri - Get the result from image.webPath
      // CameraResultType.Base64 - Get the result from image.base64String
      // CameraResultType.DataUrl - Get the result from image.dataUrl
       this.imageSrc = image.webPath
      //  imageElement.src = imageUrl;
    }

  }
}
</script>