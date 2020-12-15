<template>
  <div>
    Input is Fixed 1920 1080
    <div class="header">
      <button @click="start">Start</button>
      <button @click="end">End</button>
      <select id="resolution">
        <option value="360p">360p</option>
        <option value="480op">480p</option>
        <option value="720p">720p</option>
        <option value="1080p">1080p</option>
      </select>
      <select id="canvas-style">
        <option value="2d">2d</option>
        <option value="3d">3d</option>
      </select>
    </div>
    <hr />
    <h2>video</h2>
    <video
      id="video"
      style="width:500px; height:500px;"
      :srcObject.prop="currentStream"
    ></video>
    <hr />
    <h2>canvas</h2>
    <canvas id="canvas_3d"></canvas>
    <canvas id="canvas_2d"></canvas>
  </div>
</template>

<script>
import * as THREE from "three"
export default {
  name: "TestThree",
  data() {
    return {
      currentStream: null,
      renderer: null,
      scene: null,
      camera: null,
      texture: null,

      width: 640,
      height: 360,

      mediaRecorder: null,
      recordedChunks: []
    }
  },
  methods: {
    async start() {
      console.log("스타트")

      await this.getMedia({
        audio: true,
        video: { width: 1920, height: 1080 }
      })

      let e = document.getElementById("resolution")

      const canvas_style = document.getElementById("canvas-style").value
      this.setRes(e.value)

      const video = document.getElementById("video")
      video.play()
      const options = { mimeType: "video/webm; codecs=vp9" }

      if (canvas_style === "2d") {
        this.start2D()
        const canvas_2d = document.getElementById("canvas_2d")
        const stream = canvas_2d.captureStream()
        this.mediaRecorder = new MediaRecorder(stream, options)
        this.mediaRecorder.ondataavailable = this.handleDataAvailable
        this.mediaRecorder.start()

        //2d
      } else if (canvas_style === "3d") {
        //3d
        this.threeRender(video)

        const stream = this.renderer.domElement.captureStream(30)

        this.mediaRecorder = new MediaRecorder(stream, options)
        this.mediaRecorder.ondataavailable = this.handleDataAvailable
        this.mediaRecorder.start()
      }
    },
    start2D() {
      const canvas_2d = document.getElementById("canvas_2d")
      const video = document.getElementById("video")
      this.draw(canvas_2d, video, 0, 0, this.width, this.height)

      canvas_2d.width = this.width
      canvas_2d.height = this.height
    },
    draw(canvas_2d, video, dx, dy, width, heigth) {
      const ctx = canvas_2d.getContext("2d")
      ctx.drawImage(video, dx, dy, width, heigth)
      setTimeout(() => {
        this.draw(canvas_2d, video, dx, dy, width, heigth)
      }, 25)
    },
    end() {
      if (this.mediaRecorder) {
        this.mediaRecorder.stop()
      }
      this.recordedChunks = []

      console.log("end")
    },
    setRes(selectedRes) {
      switch (selectedRes) {
        case "360p":
          this.width = 640
          this.height = 360
          break
        case "480p":
          this.width = 854
          this.height = 480
          break
        case "720p":
          this.width = 1280
          this.height = 720
          break
        case "1080p":
          this.width = 1920
          this.height = 1080
          break
      }
    },

    async getMedia(constraints) {
      try {
        this.currentStream = await navigator.mediaDevices.getDisplayMedia(
          constraints
        )
        console.log(this.currentStream)
        /* use the stream */
      } catch (err) {
        /* handle the error */
        console.error(err)
      }
    },

    threeRender(video) {
      this.renderer = new THREE.WebGLRenderer({
        canvas: this.canvas3d,
        powerPreference: "high-performance"
      })

      this.renderer.setSize(this.width, this.height)

      // load a texture, set wrap mode to repeat
      this.texture = new THREE.VideoTexture(video)
      this.texture.minFilter = THREE.LinearFilter
      const material = new THREE.MeshBasicMaterial({
        map: this.texture
      })
      const geometry = new THREE.PlaneGeometry(272, 153)
      const plane = new THREE.Mesh(geometry, material)

      this.scene = new THREE.Scene()
      this.scene.add(plane)

      this.camera = new THREE.PerspectiveCamera(
        75,
        this.width / this.height,
        99,
        100
      )
      this.camera.position.z = 100

      //render the scene

      this.render()
    },
    render() {
      requestAnimationFrame(this.render)
      this.texture.needsUpdate = true
      this.renderer.render(this.scene, this.camera)
    },
    handleDataAvailable(event) {
      console.log("data-available")
      if (event.data.size > 0) {
        this.recordedChunks.push(event.data)
        console.log(this.recordedChunks)
        this.download()
      } else {
        console.log("nononono")
      }
    },
    download() {
      var blob = new Blob(this.recordedChunks, {
        type: "video/webm"
      })
      var url = URL.createObjectURL(blob)
      var a = document.createElement("a")
      document.body.appendChild(a)
      a.style = "display: none"
      a.href = url
      a.download = "test.webm"
      a.click()
      window.URL.revokeObjectURL(url)
    }
  },
  async mounted() {
    this.canvas3d = document.getElementById("canvas_3d")
  }
}
</script>

<style>
.header {
  display: flex;
}
</style>
