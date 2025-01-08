<template>
<!--  <a-scene embedded arjs>-->
<!--    &lt;!&ndash; مارکر Hiro &ndash;&gt;-->
<!--    <a-marker-camera preset="hiro"></a-marker-camera>-->

<!--    &lt;!&ndash; مدل عینک &ndash;&gt;-->
<!--    <a-entity-->
<!--        position="0 1.5 -6"-->
<!--        scale="2 2 2"-->
<!--        rotation="0 0 0"-->
<!--        :gltf-model="modelUrl"-->
<!--    ></a-entity>-->

<!--  </a-scene>-->
  <div style="margin: 0;padding: 0">
    <!-- ویدیو از دوربین کاربر -->
    <video ref="videoElement" width="720" height="560" autoplay muted></video>
    <!-- Canvas برای رسم نتایج تشخیص چهره -->
    <canvas ref="canvasElement" width="720" height="560" style="position: absolute;"></canvas>
  </div>
</template>

<script>
import * as faceapi from 'face-api.js';

export default {
  name: 'FaceApiExample',
  data() {
    return {
      modelsLoaded: false,
    };
  },
  mounted() {
    this.loadModels(); // بارگذاری مدل‌ها وقتی کامپوننت mount می‌شود
  },
  methods: {
    async loadModels() {
      const MODEL_URL = '/models/faceApi'; // مسیر مدل‌ها باید درست باشد (پوشه models در public قرار گیرد)

      try {
        // بارگذاری مدل‌ها
        await faceapi.nets.ssdMobilenetv1.loadFromUri(MODEL_URL);
        await faceapi.nets.faceLandmark68Net.loadFromUri(MODEL_URL);
        await faceapi.nets.faceRecognitionNet.loadFromUri(MODEL_URL);

        this.modelsLoaded = true; // وضعیت بارگذاری مدل‌ها
        console.log('Models loaded successfully');
        await this.startVideo(); // شروع پخش ویدیو پس از بارگذاری مدل‌ها
      } catch (error) {
        console.error('Error loading models:', error);
      }
    },

    async startVideo() {
      const video = this.$refs.videoElement;
      const canvas = this.$refs.canvasElement;

      // مطمئن می‌شویم که canvas به درستی با ویدیو مطابقت دارد
      faceapi.matchDimensions(canvas, video);

      video.srcObject = await navigator.mediaDevices.getUserMedia({video: {}});

      video.onplay = () => {
        this.detectFace(video, canvas);
      };
    },

    async detectFace(video, canvas) {
      if (!this.modelsLoaded) {
        console.error('Models are not loaded yet');
        return;
      }

      // انجام تشخیص چهره
      const detections = await faceapi.detectAllFaces(video)
          .withFaceLandmarks()
          .withFaceDescriptors();

      // رسم تشخیص‌ها روی canvas
      faceapi.draw.drawDetections(canvas, detections);
      faceapi.draw.drawFaceLandmarks(canvas, detections);
      // faceapi.draw.drawFaceDescriptors(canvas, detections);
      const descriptors = detections.map(d => d.descriptor);
      console.log(descriptors); // نمایش ویژگی‌های صورت

    },
  },
};
</script>

<style scoped>
/* استایل برای canvas */
canvas {
  z-index: 1;
  position: absolute;
  top: 3rem;
  left: 5rem;
}
video {
  z-index: 0;
  position: relative;
  margin: 0;
  padding: 0;
  top: 0;
  left: 0;
}
</style>
