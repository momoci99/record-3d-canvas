<template>
  <div class="hello">
    <video
      autoplay
      style="width:500px; height:500px;"
      :srcObject.prop="currentStream"
    ></video>
    <button @click="start">Start</button>
    <button @click="stop">STOP</button>
  </div>
</template>

<script>
import MSR from "../../MediaStreamRecorder";
export default {
  name: "HelloWorld",
  props: {
    msg: String
  },
  data() {
    return {
      currentStream: null,
      msr: null
    };
  },
  methods: {
    async getMedia(constraints) {
      try {
        this.currentStream = await navigator.mediaDevices.getUserMedia(
          constraints
        );
        console.log(this.currentStream);
        /* use the stream */
      } catch (err) {
        /* handle the error */
        console.error(err);
      }
    },
    async recording() {
      console.log(MSR);
      this.msr = new MSR.MultiStreamRecorder(this.currentStream, {
        video: {
          width: 1280,
          height: 720
        }
      });

      // this.msr.start();
    },
    start() {
      this.msr.start();
    },
    stop() {
      this.msr.stop();
    }
  },

  async mounted() {
    await this.getMedia({
      audio: true,
      video: { width: 1280, height: 720 }
    });
    const canvas3d = document.getElementById("canvas_3d");
    const srcVideo = document.getElementById("v1");
    this.recording();
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
