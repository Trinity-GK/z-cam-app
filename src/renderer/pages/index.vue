<template>
  <div class="container">
    <button class="btn btn-primary" type="button" @click="getCamerasFromNetwork">FETCH CAMS</button>
    <!--    <button type="button" @click="getSessions">CONTROL</button>-->
    <b-modal id="video-modal" title="BootstrapVue">
      <div class="modal-content">
        <img :src="`http://${currentStreamIP}/mjpeg_stream`"/>
      </div>
    </b-modal>
    <div class="row">
      <input type="text" v-model="addIP" placeholder="xxx.xxx.xxx.xxx"/>
      <button class="btn btn-success" type="button" @click="addCamera">ADD CAMERA</button>
      <table class="table">
        <thead>
        <tr>
          <th>#</th>
          <th>IP</th>
          <th>Model</th>
          <th>SN</th>
          <th>Status</th>
          <th>Preview</th>
          <th>Actions</th>
        </tr>
        </thead>
        <tbody>
        <tr v-for="(camera, index) in cameras">
          <th scope="row">{{ index + 1 }}</th>
          <td>{{ camera.eth_ip }}</td>
          <td>{{ camera.model }}</td>
          <td>{{ camera.sn }}</td>
          <td>{{ status(camera.status) }}</td>
          <td>
            <img style="width: 32px; height: 24px" :src="`http://${camera.eth_ip}/mjpeg_stream`"
                 @click="playVideo(camera.eth_ip);"/>
          </td>
          <td>
            <button type="button" class="btn btn-success" @click="startRecording(index)">START</button>
            <button type="button" class="btn btn-warning" @click="stopRecording(index)">STOP</button>
            <button type="button" class="btn btn-danger" @click="turnOff(index)">OFF</button>
          </td>
        </tr>
        </tbody>
      </table>

    </div>
    <div class="row" v-for="(camera, index) in cameras">
      <div class="card" style="width: 50%;">
        <img class="card-img-top" :src="`http://${camera.eth_ip}/mjpeg_stream`">
        <div class="card-img-overlay">
          <h5 class="card-title text-white">{{camera.minutesRemaining}}m</h5>
        </div>
        <div class="card-body">
          <h5 class="card-title">{{ camera.eth_ip }}</h5>
          <p class="card-text">{{ status(camera.status) }}</p>
          <button class="btn btn-success" @click="startRecording(index)">RECORD</button>
          <button class="btn btn-warning" @click="stopRecording(index)">STOP</button>
          <button class="btn btn-danger" @click="turnOff(index)">OFF</button>
          <button class="btn btn-dark" @click="reboot(index)">REBOOT</button>
        </div>
        <div class="card-body">
          <CameraMenu />
        </div>
        <div class="card-body">
          <b-tabs content-class="mt-3" justified>
            <b-tab title="Record" :active="currentMode(index, ['rec','rec_ing','rec_paused '])">
              <button type="button" class="btn btn-primary btn-block" @click="switchMode(index, 'to_rec')">SWITCH TO
                RECORDING MODE
              </button>
            </b-tab>
            <b-tab title="Timelapse" :active="currentMode(index, ['cap_tl_idle', 'cap_tl_ing'])">
              <button type="button" class="btn btn-primary btn-block" @click="switchMode(index, 'to_rec_tl')">SWITCH TO
                TIMELAPSE MODE
              </button>
            </b-tab>
            <b-tab title="Playback" :active="currentMode(index, ['pb', 'pb_ing', 'pb_paused'])">
              <button type="button" class="btn btn-primary btn-block" @click="switchMode(index, 'to_pb')">SWITCH TO
                PLAYBACK MODE
              </button>
            </b-tab>
          </b-tabs>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import {mapMutations} from 'vuex';
import {networkInterfaces} from 'os';
import IPCIDR from 'ip-cidr';
import ping from 'ping';
import axios from 'axios';
import CameraMenu from "../components/CameraMenu";

export default {
  components: {CameraMenu},
  data() {
    return {
      addIP: '',
      cameras: [],
      timer: '',
      currentStreamIP: ''
    }
  },
  created() {
    this.timer = setInterval(() => {
      this.fetchStatus()
    }, 5000)
  },
  methods: {
    currentMode(index, modes) {
      return modes.includes(this.cameras[index].status)
    },
    switchMode(index, mode) {
      switch (mode) {
        case 'to_rec_tl':
          this.cameras[index].enable_video_tl = !!!this.cameras[index].enable_video_tl;
          axios.get(`http://${this.cameras[index].eth_ip}/ctrl/set?enable_video_tl=${this.cameras[index].enable_video_tl}`)
          axios.get(`http://${this.cameras[index].eth_ip}/ctrl/mode?action=${mode}`)

          break;
        case 'to_rec':
        case 'to_pb':
          axios.get(`http://${this.cameras[index].eth_ip}/ctrl/mode?action=${mode}`)
          break;
      }
    },
    addCamera() {
      axios.get(`http://${this.addIP}/info`).then(info => {
        // at least returned an object
        if (typeof info.data === 'object' && info.data.hasOwnProperty('sn')) {
          console.log(info);
          const exists = this.cameras.findIndex(c => c.eth_ip === info.data.eth_ip);

          if (exists >= 0) {
            this.cameras[exists] = info.data;
          } else {
            axios.get(`http://${this.addIP}/ctrl/session`).then(control => {
              console.log('GOT CONTROL', control)
              this.cameras.push(info.data);
              this.addIP = '';
            })
          }
        }
      }).catch(err => {
        // console.log(err);
      })
    },
    stopRecording(index) {
      axios.get(`http://${this.cameras[index].eth_ip}/ctrl/rec?action=stop`)
    },
    turnOff(index) {
      axios.get(`http://${this.cameras[index].eth_ip}/ctrl/shutdown`);
      this.cameras = this.cameras.filter(c => c.eth_ip !== this.cameras[index].eth_ip);
    },
    reboot(index) {
      axios.get(`http://${this.cameras[index].eth_ip}/ctrl/reboot`);
      this.cameras = this.cameras.filter(c => c.eth_ip !== this.cameras[index].eth_ip);

    },
    startRecording(index) {
      axios.get(`http://${this.cameras[index].eth_ip}/ctrl/rec?action=start`)
    },
    async getCamerasFromNetwork() {
      const interfaces = networkInterfaces();

      let addresses = [];

      for (let ifc in interfaces) {

        let filter = ['VMware', 'Loopback', 'vEthernet'];

        const filtered = filter.some(f => ifc.includes(f))

        if (!filtered) {
          for (let e of interfaces[ifc]) {
            if (e.family === 'IPv4' && e.internal === false && e.cidr.split('/')[1] === '24') {
              const cidr = new IPCIDR(e.cidr);

              addresses = [...addresses, ...cidr.toArray()]
            }
          }
        }
      }

      console.log('CHECKING', addresses.length)

      for (let address of addresses) {
        console.log(`Checking ${address}`);

        try {
          const pingRes = await ping.promise.probe(address);

          if (!!!pingRes.alive) continue;

          const infoRes = await axios.get(`http://${address}/info`, {timeout: 200});

          if (typeof infoRes.data === 'object' && infoRes.data.hasOwnProperty('sn')) {
            console.log(infoRes);
            const exists = this.cameras.findIndex(c => c.eth_ip === infoRes.data.eth_ip);


            if (exists >= 0) {
              this.cameras[exists] = infoRes.data;
            } else {
              await axios.get(`http://${address}/ctrl/session`)
              this.cameras.push(infoRes.data);
            }
          }
        } catch (e) {

        }
      }
    },
    getSessions() {
      // for(let camera of this.cameras) {
      //   axios.get(`http://${camera.eth_ip}/ctrl/session`).then(res => {
      //     console.log(res);
      //   })
      // }
    },
    async fetchStatus() {
      const cameras = await Promise.all(this.cameras.map(async c => {
        try {
          const {data} = await axios.get(`http://${c.eth_ip}/ctrl/mode?action=query`);
          console.log(data);

          const timeRemaining = await axios.get(`http://${c.eth_ip}/ctrl/rec?action=remain`);
          console.log(timeRemaining.data);
          c.minutesRemaining = timeRemaining.data.msg;
          c.status = data.msg;
        } catch (e) {

        }

        return c;
      }));

      this.cameras = cameras;
    },
    status(s) {
      const statusMap = {
        rec: 'Record Mode',
        rec_ing: 'Recording',
        rec_paused: 'Recording Paused',

        pb: 'Playback Mode',
        pb_ing: 'Playing Back',
        pb_paused: 'Playing Paused',

        rec_tl: 'Timelapse Mode',
        rec_tl_ing: 'Timelapse Recording',
        rec_tl_idle: 'Timelapse Idle',

        standby: 'Standby'
      }

      return statusMap[s];
    },
    playVideo(ip) {
      this.currentStreamIP = ip;
      this.$bvModal.show('video-modal')
    },
    getCameraSettings(index) {

    }
  },
  beforeDestroy() {
    clearInterval(this.timer)
  }
}
</script>

<style>

.modal-dialog {
  justify-content: center !important;
}

.modal-content {
  width: initial !important;
}

.container {
  width: 100%;
}
</style>
