<template>
  <div id="app">
    <div class="main-cam mr-20">
      <web-cam
        ref="webcam"
        width="400px"
        height="400px"
        :device-id="deviceId"
        @started="onStarted"
        @stopped="onStopped"
        @error="onError"
        @cameras="onCameras"
        @camera-change="onCameraChange" />

      <div class="select-device mb-20">
        <select v-model="camera">
            <option>-- Select Device --</option>
            <option
                v-for="device in devices"
                :key="device.deviceId"
                :value="device.deviceId"
            >{{ device.label }}</option>
        </select>
      </div>

      <div class="cam-controls">
        <button type="button" class="btn btn-primary mr-20" @click="onCapture">Capture Photo</button>
        <button type="button" class="btn btn-danger mr-20" @click="onStop">Stop Camera</button>
        <button type="button" class="btn btn-success" @click="onStart">Start Camera</button>
      </div>

      <div class="card-details">
        <h2>Card Details</h2>

        <p> confidence: {{ get(imageData, 'result[0].confidence') }} </p>
        <p> id: {{ get(imageData, 'result[0].id') }} </p>
        <p> price: {{ get(imageData, 'result[0].price') }} </p>

        <p> manufacturer: {{ get(imageData, 'result[0].metadata.manufacturer') }} </p>
        <p> player: {{ get(imageData, 'result[0].metadata.player') }} </p>
        <p> sport: {{ get(imageData, 'result[0].metadata.sport') }} </p>
        <p> team: {{ get(imageData, 'result[0].metadata.team') }} </p>
      </div>
    </div>
    <div class="captured-image">
      <h2>Captured Image</h2>
      <figure class="figure">
          <img :src="img" class="img-responsive" />
      </figure>
    </div>
  </div>
</template>

<script>
import { WebCam } from "vue-web-cam";
import Axios from 'axios'
import get from 'lodash/get'

export default {
  name: 'App',
  data() {
    return {
        img: null,
        camera: null,
        deviceId: null,
        devices: [],
        base64img: '',
        imageData: '',
    };
  },
  computed: {
      device: function() {
          return this.devices.find(n => n.deviceId === this.deviceId);
      }
  },
  watch: {
      camera: function(id) {
          this.deviceId = id;
      },
      devices: function() {
          // Once we have a list select the first one
          const [first] = this.devices;
          if (first) {
              this.camera = first.deviceId;
              this.deviceId = first.deviceId;
          }
      }
  },
  methods: {
      onCapture() {
          this.img = this.$refs.webcam.capture();
          const dataImg = this.img.replace(/^data:image\/[a-z]+;base64,/, "");
          Axios.post('https://iz1ywuf6z7.execute-api.ap-southeast-2.amazonaws.com/prod/predict', {data: dataImg}).then((res) => {
            this.imageData = res.data
          })
      },
      onStarted(stream) {
          console.log("On Started Event", stream);
      },
      onStopped(stream) {
          console.log("On Stopped Event", stream);
      },
      onStop() {
          this.$refs.webcam.stop();
      },
      onStart() {
          this.$refs.webcam.start();
      },
      onError(error) {
          console.log("On Error Event", error);
      },
      onCameras(cameras) {
          this.devices = cameras;
          console.log("On Cameras Event", cameras);
      },
      onCameraChange(deviceId) {
          this.deviceId = deviceId;
          this.camera = deviceId;
          console.log("On Camera Change Event", deviceId);
      },
      get,
  },
  components: {
    WebCam
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  display: flex;
}
.main-cam {
  display: flex;
  align-items: center;
  flex-direction: column;
}

.mr-20 {
  margin-right: 20px;
}

.mb-20 {
  margin-bottom: 20px
}

.card-details {
  display: flex;
  flex-direction: column;
  text-align: left;
}
</style>
