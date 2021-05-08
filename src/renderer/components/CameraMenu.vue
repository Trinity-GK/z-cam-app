<template>
  <div class="container-fluid">
    <div class="row justify-content-md-center">
      <div class="col-4 d-flex flex-column align-items-center">
        <BIconCameraVideoFill font-scale="3" />
        Record
      </div>
      <div class="col-4 d-flex flex-column align-items-center">
        <BIconFilm font-scale="3" />
        Video
      </div>
      <div class="col-4 d-flex flex-column align-items-center">
        <BIconVolumeUpFill font-scale="3" />
        Audio
      </div>
      <div class="col-4 d-flex flex-column align-items-center">
        <BIconSquareHalf font-scale="3" />
        Exposure
      </div>
      <div class="col-4 d-flex flex-column align-items-center">
        <BIconBrightnessHigh font-scale="3" />
        WB
      </div>
      <div class="col-4 d-flex flex-column align-items-center">
        <BIconPlusSquare font-scale="3" />
        Focus
      </div>
      <div class="col-4 d-flex flex-column align-items-center">
        <BIconCardImage font-scale="3" />
        Image
      </div>
      <div class="col-4 d-flex flex-column align-items-center">
        <BIconLink45deg font-scale="3" />
        Connect
      </div>
      <div class="col-4 d-flex flex-column align-items-center">
        <BIconGearFill font-scale="3" />
        System
      </div>
    </div>
  </div>
</template>

<script>
import {
  BIconCameraVideoFill,
  BIconFilm,
  BIconVolumeUpFill,
  BIconBrightnessHigh,
  BIconPlusSquare,
  BIconCardImage,
  BIconGearFill,
  BIconSquareHalf,
  BIconLink45deg
} from 'bootstrap-vue'

export default {
  name: "CameraMenu",
  components: {
    BIconCameraVideoFill,
    BIconFilm,
    BIconVolumeUpFill,
    BIconBrightnessHigh,
    BIconPlusSquare,
    BIconCardImage,
    BIconGearFill,
    BIconSquareHalf,
    BIconLink45deg
  },
  data() {
    return {
      submenu: {
        video: {
          key: 'video'
        },
        focus: {},
        exposure: {},
        wb: {},
        image: {},
        audio: {},
      }
    }
  },
  methods: {

  }
}
</script>

<style scoped>
[class*="col-"]{
  border: 2px solid black;
  padding: 1em;
}
</style>
