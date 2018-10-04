<template>
  <div class='camera' ref="wrapper">

    <div class="camera-wrapper">
      <video ref="camera" v-show="!blob || (capture === 'video' && isAlreadyRecorded) || (capture !== 'photo' && !isAlreadyRecorded)" id='preview' width="100%" height="auto" autoPlay></video>
      <img ref="photo" v-if="capture === 'photo'" v-show="blob" alt="photo" width="100%" height="auto">
    </div>

    <select id="camera-select" v-if="devices.length && !isRecording && !isAlreadyRecorded" v-model="selected" @change="cameraChanged">
      <option v-for="(device, i) in devices" :key="i" :value="device">{{device.label || device.kind}}</option>
    </select>

    <div  v-if="isRecording" class="counter">
      <slot name="counter">
        <div class="content">
          <i class="fa fa-circle" aria-hidden="true" style="font-size: 0.5rem;"></i>
          {{recordingTime}}
        </div>
      </slot>
    </div>

    <div class="buttons-wrapper">

      <div v-if="!isRecording && !isAlreadyRecorded" class="record-button" @click="action">
        <i class="fa fa-camera" aria-hidden="true"></i>
      </div>

      <div v-if="isRecording" class="record-button" @click="stopRecording">
        <i class="fa fa-stop" aria-hidden="true"></i>
      </div>

      <div :class="onReadyButtonsClass">
        <div v-if="!isRecording && isAlreadyRecorded" class="reset-button" :class="{'capture-photo': capture === 'photo'}" @click="resetStream">
          <i class="fa fa-repeat" aria-hidden="true"></i>
        </div>
        <div v-if="!isRecording && isAlreadyRecorded" class="upload-button" :class="{'capture-photo': capture === 'photo'}" @click="onReady">
          <i class="fa fa-cloud-upload" aria-hidden="true"></i>
        </div>
      </div>

    </div>

  </div>
</template>

<script>
export default {
  name: 'camera-component',

  props: {
    capture: {
      type: String,
      default: 'video'
    }
  },

  computed: {
    recordingTime () {
      const minutes = Math.floor(this.counter / 60)
      const seconds = this.counter % 60 < 10 ? `0${this.counter % 60}` : this.counter % 60
      return minutes + ':' + seconds
    },
    onReadyButtonsClass () {
      let classes = 'on-ready-buttons'
      if (this.capture === 'video') {
        if (this.paused) {
          classes += ' paused'
        }
        if (this.smallPreview) {
          classes += ' smallPreview'
        }
      } else {
        classes += ' capture-photo'
      }
      return classes
    }
  },

  methods: {
    action () {
      const actions = {
        photo: this.takePhoto,
        video: this.startRecording
      }
      const action = actions[this.capture]
      action()
    },

    takePhoto () {
      this.$emit('onPhoto')
      const photo = this.$refs.photo
      navigator.mediaDevices.getUserMedia(this.constraints).then(stream => {
        const mediaStreamTrack = stream.getVideoTracks()[0]
        const imageCapture = new ImageCapture(mediaStreamTrack)
        imageCapture.takePhoto()
          .then(blob => {
            this.blob = blob
            this.isAlreadyRecorded = true
            photo.src = URL.createObjectURL(blob)
            photo.onload = () => { URL.revokeObjectURL(this.src) }
          })
      })
    },

    startRecording () {
      this.$emit('onStart')
      this.time()
      let preview = document.getElementById('preview')
      const startRecording = (stream) => {
        this.recordGlobal = new MediaRecorder(stream)
        this.recordGlobal.start()
      }
      navigator.mediaDevices.getUserMedia(this.constraints).then(stream => {
        preview.setAttribute('autoplay', true)
        preview.removeAttribute('controls')
        preview.srcObject = stream
        preview.captureStream = preview.captureStream || preview.mozCaptureStream
        return new Promise(resolve => {
          preview.onplaying = resolve
          resolve()
        })
      }).then(() => startRecording(preview.captureStream()))
    },

    stopRecording () {
      this.$emit('onStop')
      this.isAlreadyRecorded = true
      this.paused = true
      this.clearInterval()
      let preview = document.getElementById('preview')
      let data = []
      if (this.recordGlobal.state === 'recording') {
        this.recordGlobal.stop()
      }
      this.recordGlobal.ondataavailable = event => {
        data.push(event.data)
        let recordedBlob = new Blob(data, {type: 'video/webm'})
        this.blob = recordedBlob
        preview.srcObject = null
        preview.src = URL.createObjectURL(recordedBlob)
        preview.removeAttribute('autoplay')
        preview.setAttribute('controls', true)
      }
    },

    resetStream () {
      this.$emit('onReset')
      this.blob = null
      let preview = document.getElementById('preview')
      if (this.capture === 'photo') {
        this.isAlreadyRecorded = false
        return
      }
      navigator.mediaDevices
        .getUserMedia(this.constraints).then(stream => {
          preview.setAttribute('autoplay', true)
          preview.removeAttribute('controls')
          preview.srcObject = stream
          preview.captureStream = preview.captureStream || preview.mozCaptureStream
          this.isAlreadyRecorded = false
        })
    },

    onReady () {
      this.$emit('onReady', this.blob)
    },

    time () {
      this.isRecording = true
      this.timeCounter = setInterval(() => {
        this.counter = this.counter + 1
      }, 1000)
    },

    clearInterval () {
      clearInterval(this.timeCounter)
      this.isRecording = false
      this.counter = 0
      this.timeCounter = null
    },

    cameraChanged () {
      this.$emit('onCameraChanged')
      const preview = document.getElementById('preview')
      const videoConstraints = {}
      videoConstraints.deviceId = { exact: this.selected.deviceId }
      this.constraints = {
        video: videoConstraints,
        audio: false
      }
      navigator.mediaDevices
        .getUserMedia(this.constraints)
        .then(stream => { preview.srcObject = stream })
        .catch(err => console.error(err))
    },

    handleWindowResize () {
      this.smallPreview = this.$refs.wrapper.offsetWidth < 740
    },

    handleOnPlay () {
      this.paused = false
      if (this.isAlreadyRecorded) {
        this.$emit('onPlay')
      }
    },

    handleOnPause () {
      this.paused = true
      this.$emit('onPause')
    }
  },

  mounted () {
    const camera = this.$refs.camera
    window.addEventListener('resize', this.handleWindowResize)
    this.handleWindowResize()
    camera.addEventListener('play', this.handleOnPlay)
    camera.addEventListener('pause', this.handleOnPause)
    this.constraints = {video: true}
    const gotDevices = (mediaDevices) => {
      this.devices = JSON.parse(JSON.stringify(mediaDevices)).filter(d => d.kind === 'videoinput')
      this.selected = this.devices[0]
      let videoConstraints = {}
      videoConstraints.deviceId = { exact: this.selected.deviceId }
      this.constraints = {
        video: videoConstraints,
        audio: false
      }
    }
    navigator.mediaDevices
      .getUserMedia(this.constraints)
      .then(stream => {
        camera.srcObject = stream
        return navigator.mediaDevices.enumerateDevices()
      })
      .then(gotDevices)
      .catch(error => {
        console.error(error)
      })
  },

  beforeDestroy () {
    const camera = this.$refs.camera
    window.removeEventListener('resize', this.handleWindowResize)
    camera.removeEventListener('play', this.handleOnPlay)
    camera.removeEventListener('pause', this.handleOnPause)
  },

  data () {
    return {
      paused: false,
      smallPreview: false,
      select: false,
      blob: null,
      devices: [],
      constraints: {},
      selected: null,
      isRecording: false,
      isAlreadyRecorded: false,
      counter: 0,
      timeCounter: null
    }
  }
}
</script>

<style lang='scss'>
@import "@/assets/css/variables.scss";

.camera {
  &:hover {
    .on-ready-buttons {
      opacity: 1;
    }
  }
}
.camera {
  position: relative;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.camera-wrapper {
  position: relative;
  width: 100%;

  video {
    position: relative;
    border-radius: 5px;
  }

  img {
    position: relative;
    border-radius: 5px;
  }

}
.counter {
  position: absolute;
  top: 1rem;
  left: 1rem;
  margin: 0 auto;
  width: fit-content;
  .content {
    display: flex;
    align-items: center;
    color: #ff6666;
    background-color: rgba(0,0,0,0.5);
    padding: 2px 10px;
    border-radius: 3px;
  }
  i {
    margin-right: .5rem;
  }
}
.buttons-wrapper {
  position: absolute;
  top: 0;
  bottom: 6px;
  left: 0;
  right: 0;
  width: 100%;
  pointer-events: none;

  i {
    margin: 0 .5rem;
    span {
      font-family: 'Avenir', Helvetica, Arial, sans-serif;
    }
  }
  .on-ready-buttons {
    display: grid;
    grid-template-columns: 1fr 1fr 1fr;
    position: relative;
    width: 60%;
    left: 0;
    right: 0;
    margin: 0 auto;
    opacity: 0;
    transition: all 1s;
    top: calc(50% - 2rem - 20px);

    &.paused {
      opacity: 1;
    }

    &.smallPreview {
      top: calc(50% - 2rem - 8px);
    }

    &.capture-photo {
      top: calc(100% - 2rem) !important;
      grid-template-columns: 1fr 1fr;
      width: 30%;
      opacity: 1;
    }
  }
  .cancel-button {
    pointer-events: auto;
    position: absolute;
    cursor: pointer;
    width: 1.5rem;
    height: 1.5rem;
    top: 1rem;
    left: 1rem;
    .icon {
      position: absolute;
      top: 0;
      left: .75rem;
      transform: rotate(45deg);
      width: 1px;
      height: 1.5rem;
      background-color: black;
      &::before {
        content: '';
        position: absolute;
        height: 1px;
        width: 1.5rem;
        top: .75rem;
        left: -.75rem;
        background-color: black;
      }
    }
  }
  .reset-button {
    position: relative;
    bottom: 0;
    left: 0;
    right: 0;
    display: flex;
    justify-content: center;
    align-items: center;
    height: 4rem;
    width: 4rem;
    margin: 0 auto;
    color: white;
    background-color: #ff6666;
    border-radius: 50%;
    cursor: pointer;
    pointer-events: auto;
    opacity: 0.9;

    &.capture-photo {
      opacity: 1;
      box-shadow: 0 1px 3px rgba(0, 0, 0, 0.4);
    }
  }
  .upload-button {
    transition: all 0.5s;
    grid-column-start: 3;
    position: relative;
    bottom: 0;
    left: 0;
    right: 0;
    display: flex;
    justify-content: center;
    align-items: center;
    height: 4rem;
    width: 4rem;
    margin: 0 auto;
    color: white;
    background-color: #29c57f;
    border-radius: 50%;
    cursor: pointer;
    pointer-events: auto;
    opacity: 0.9;

    &.capture-photo {
      grid-column-start: 2;
      opacity: 1;
      box-shadow: 0 1px 3px rgba(0, 0, 0, 0.4);
    }
  }
  .record-button {
    position: absolute;
    bottom: -2rem;
    left: 0;
    right: 0;
    display: flex;
    justify-content: center;
    align-items: center;
    height: 4rem;
    width: 4rem;
    margin: 0 auto;
    color: white;
    background-color: #ff6666;
    border-radius: 50%;
    box-shadow: 0 1px 3px rgba(0, 0, 0, 0.4);
    cursor: pointer;
    pointer-events: auto;
  }
}
#camera-select {
  position: absolute;
  width: fit-content;
  top: 1rem;
  right: 1rem;
  z-index: 1;
}

</style>
