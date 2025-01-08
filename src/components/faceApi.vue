<script setup>
import * as faceapi from 'face-api.js'
import {useTemplateRef} from "vue";

const video = useTemplateRef('video')

function startVideo() {
  navigator.getUserMedia(
      {video: {}},
      stream => video.srcObject = stream,
      err => console.error(err))
}

// startVideo()

</script>

<template>
  <div class="body">
    <button @click="startVideo">start</button>
    <p>jkgjgkj</p>
    <video ref="video" id="video" width="720" height="560" autoplay muted></video>
  </div>
</template>

<style scoped>
.body {
  padding: 0;
  margin: 0;
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>