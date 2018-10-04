# vuejs-camera
A component that allow to record a video o take pictures in your website with the HTML5 API.

![demo gif](https://github.com/juandiegombr/vuejs-camera/blob/master/demo.gif?raw=true)


## DEMO

[https://juandiegombr.github.io/vuejs-camera/]( https://juandiegombr.github.io/vuejs-camera/)


## Installation
```
yarn add vuejs-camera
npm install --save vuejs-camera
```

```
import VueCamera from 'vuejs-camera'

export default {
  components: {
    VueCamera
  }
}
```

```
<VueCamera />
```

## Props

### capture

- Type: `String`
- Default: `video` 
- Options: `video` | `photo`

Allow you to capture video or photo.

### Events

### onReady

When the upload button has been pressed. The Blob (Video or Photo) is sended in the payload.

### onPhoto

The photo has been taken.

### onStart

It starts to record the video.

### onStop

The record has been stoped.

### onReset

The video or photo is reseted.

### onCameraChanged

The source camera has been changed.

### onPlay

The video has been played.

### onPause

The video has been paused.
