# vuejs-camera 
A component that allow to record a video o take pictures in your website with the HTML5 API.

![npm (scoped)](https://img.shields.io/npm/v/@mrjeffapp/vuejs-camera.svg?style=flat-square)

![demo gif](https://github.com/juandiegombr/vuejs-camera/blob/master/demo.gif?raw=true)


## DEMO

[https://mrjeffapp.github.io/vuejs-camera/]( https://mrjeffapp.github.io/vuejs-camera/)


## Installation
```
yarn add @mrjeffapp/vuejs-camera
npm install --save @mrjeffapp/vuejs-camera
```

```
// Global - In your main.js
import '@mrjeffapp/vuejs-camera'

// Local - In your Component.vue
import VueCamera from '@mrjeffapp/vuejs-camera'

export default {
  components: {
    VueCamera
  }
}
```
```
// Add in your index.html
<script src="https://use.fontawesome.com/e9b7441153.js"></script>
```

```
// Enjoy it!
<VueCamera/>
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
