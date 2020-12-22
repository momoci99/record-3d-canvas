<template>
	<div>
		<div class="header">
			<button @click="start">Start</button>
			<button @click="end">End</button>
			<button @click="swap">swap</button>
			<select id="resolution">
				<option value="360p">360p</option>
				<option value="480op">480p</option>
				<option value="720p">720p</option>
				<option value="1080p">1080p</option>
				<option value="1440p">1440p</option>
				<option value="2160p">2160p</option>
			</select>
			<select id="canvas-style">
				<option value="2d">2d</option>
				<option value="3d">3d</option>
			</select>
		</div>
		<hr />


		<video
			id="video-sample-4k"
			src="../../public/fireworks.mp4"
			autoplay
			muted
			loop
			class="video-conf"
		></video>
		<!-- <h2>conference</h2>
		<video
			id="video-sample-4k"
			src="../../public/fireworks.mp4"
			autoplay
			muted
			loop
			class="video-conf"
		></video>
		<video
			src="../../public/fireworks.mp4"
			autoplay
			muted
			loop
			class="video-conf"
		></video>
		<video
			src="../../public/fireworks.mp4"
			autoplay
			muted
			loop
			class="video-conf"
		></video>
		<video
			src="../../public/fireworks.mp4"
			autoplay
			muted
			loop
			class="video-conf"
		></video>
		<video
			src="../../public/fireworks.mp4"
			autoplay
			muted
			loop
			class="video-conf"
		></video> -->

		<hr />
		<h2>current stream</h2>
		<video
			:srcObject.prop="currentStream"
			autoplay
			id="video"
			style="width: 500px; height: 500px"
		></video>
		<hr />
		<h2>canvas</h2>
		<canvas id="canvas_3d" style="display: none"></canvas>
		<canvas id="canvas_2d" style="display: none"></canvas>
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

			width: 1920,
			height: 1080,

			mediaRecorder: null,
			recordedChunks: [],

			camStream: null,
			screenStream: null,

			targetVideo: null,
			toggle: false,

			canvas_style: null,

			//for time slice
			timeSlice: 5000,

			timer: null,
		}
	},
	methods: {
		async start() {
			console.log("스타트")

			let e = document.getElementById("resolution")

			this.canvas_style = document.getElementById("canvas-style").value
			const selectedRes = e.value
			this.setRes(selectedRes)

			// await this.getCam({
			// 	video: { width: this.width, height: this.height },
			// 	audio: true,
			// })

			const video = document.getElementById("video")
			video.play()
			const options = { mimeType: "video/webm; codecs=vp9" }

			let recordingStream = new MediaStream()

			if (this.canvas_style === "2d") {
				this.start2D()
				const canvas_2d = document.getElementById("canvas_2d")
				const stream = canvas_2d.captureStream()
				recordingStream.addTrack(stream.getVideoTracks()[0])
			} else if (this.canvas_style === "3d") {
				this.threeRender(video)
				const stream = this.renderer.domElement.captureStream(24)
				recordingStream.addTrack(stream.getVideoTracks()[0])
			}

			//4k는 샘플영상에서 스트림을 취할생각임
			if (selectedRes === "2160p") {
				const sample_video = document.getElementById("video-sample-4k")
				this.currentStream = sample_video.captureStream()
				recordingStream = this.currentStream
			} else {
				recordingStream.addTrack(this.currentStream.getAudioTracks()[0])
			}

			this.startRecording(recordingStream, options)
			// this.timeSlicer()
		},
		start2D() {
			const canvas_2d = document.getElementById("canvas_2d")
			const video = document.getElementById("video")
			this.draw(canvas_2d, video, 0, 0, this.width, this.height)

			canvas_2d.width = this.width
			canvas_2d.height = this.height
		},
		draw(canvas_2d, video, dx, dy, width, heigth) {
			const ctx = canvas_2d.getContext("2d", { alpha: false })
			ctx.drawImage(video, dx, dy, width, heigth)
			//1초에 약 24번 drawing을 수행하여 24fps 구현
			setTimeout(() => {
				this.draw(canvas_2d, video, dx, dy, width, heigth)
			}, 40)
		},
		end() {
			if (this.mediaRecorder) {
				this.mediaRecorder.stop()
				clearInterval(this.timer)
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
				case "1440p":
					this.width = 2560
					this.height = 1440
					break

				case "2160p":
					this.width = 3840
					this.height = 2160
					break
			}
		},

		async getMedia(constraints) {
			try {
				this.screenStream = await navigator.mediaDevices.getDisplayMedia(
					constraints
				)

				this.currentStream = this.camStream
				/* use the stream */
			} catch (err) {
				/* handle the error */
				console.error(err)
			}
		},

		async getCam(constraints) {
			try {
				this.camStream = await navigator.mediaDevices.getUserMedia(constraints)
				this.currentStream = this.camStream
			} catch (err) {
				console.log(err)
			}
		},

		threeRender(video) {
			this.renderer = new THREE.WebGLRenderer({
				canvas: this.canvas3d,
				powerPreference: "high-performance",
			})

			this.renderer.setSize(this.width, this.height)

			// load a texture, set wrap mode to repeat
			this.texture = new THREE.VideoTexture(video)
			this.texture.minFilter = THREE.LinearFilter
			const material = new THREE.MeshBasicMaterial({
				map: this.texture,
			})

			//2차원 평면 생성
			// const geometry = new THREE.PlaneGeometry(this.width, this.height)
			const geometry = new THREE.PlaneGeometry(272, 153)
			const plane = new THREE.Mesh(geometry, material)

			this.scene = new THREE.Scene()
			this.scene.add(plane)

			//plane geometry라 aspect ratio 외에는 크게 의마가 없는듯???
			this.camera = new THREE.PerspectiveCamera(75, 1.77, 99, 100)
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
			try {
				console.log("data-available")
				if (event.data.size > 0) {
					this.recordedChunks.push(event.data)
					console.log(this.recordedChunks)
					this.download()
				} else {
					console.log("nononono")
				}
			} catch (e) {
				console.error(e)
			}
		},
		download() {
			var blob = new Blob(this.recordedChunks, {
				type: "video/mp4",
			})
			var url = URL.createObjectURL(blob)
			var a = document.createElement("a")
			document.body.appendChild(a)
			a.style = "display: none"
			a.href = url
			a.download =
				"test" +
				this.width +
				"x" +
				this.height +
				"_" +
				this.canvas_style +
				".mp4"
			a.click()
			window.URL.revokeObjectURL(url)
		},

		//안됨 에러남 ㅋ
		swap() {
			console.log("switch")
			console.log(this.toggle)
			const video = document.getElementById("video")

			this.currentStream = this.toggle ? this.camStream : this.screenStream
			console.log(video)
			this.toggle = !this.toggle
		},

		timeSlicer() {
			this.timer = setTimeout(() => {
				if (!this.mediaRecorder) {
					return
				}

				if (this.mediaRecorder.state === "recording") {
					this.mediaRecorder.requestData()
				}
			}, this.timeSlice)
		},
		startRecording(stream, options) {
			this.mediaRecorder = new MediaRecorder(stream, options)
			this.mediaRecorder.ondataavailable = this.handleDataAvailable
			this.mediaRecorder.start()
		},
	},
	async mounted() {
		this.canvas3d = document.getElementById("canvas_3d")
		await this.getCam({
			video: { width: this.width, height: this.height },
			audio: true,
		})
	},
}
</script>

<style>
.header {
	display: flex;
}
.video-conf {
	width: 360px;
	height: 180px;
}
</style>
