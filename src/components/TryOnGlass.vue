<template>
  <div class="video-stream">
    <video id="video" ref="video" autoplay muted></video>
    <canvas id="overlay" ref="overlay"></canvas>
  </div>
</template>

<script>
import { shallowRef, onMounted, onBeforeUnmount } from 'vue';
import * as faceapi from 'face-api.js';
import * as THREE from 'three';
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader';

export default {
  name: 'TryOnGlasses',
  setup() {
    const video = shallowRef(null);
    const overlay = shallowRef(null);
    const scene = shallowRef(null);
    const camera = shallowRef(null);
    const renderer = shallowRef(null);
    const glassesModel = shallowRef(null);
    let detectionInterval = null;

    const loadFaceApiModels = async () => {
      await faceapi.nets.tinyFaceDetector.loadFromUri('/models/faceApi');
      await faceapi.nets.faceLandmark68Net.loadFromUri('/models/faceApi');
      console.log('FaceAPI models loaded');
    };

    const loadGlassesModel = (scene) => {
      const loader = new GLTFLoader();
      loader.load(
          '/models/glasses.glb',
          (gltf) => {
            glassesModel.value = gltf.scene;
            glassesModel.value.scale.set(0.015, 0.015, 0.015); // تنظیم مقیاس
            glassesModel.value.rotation.set(0, 0, 0); // صاف کردن چرخش
            scene.value.add(glassesModel.value);
            console.log('3D glasses model loaded');
          },
          undefined,
          (error) => {
            console.error('Error loading 3D model:', error);
          }
      );
    };

    const setupThreeJS = () => {
      renderer.value = new THREE.WebGLRenderer({canvas: overlay.value, alpha: true});
      renderer.value.setSize(overlay.value.clientWidth, overlay.value.clientHeight);
      renderer.value.setPixelRatio(window.devicePixelRatio);

      scene.value = new THREE.Scene();
      camera.value = new THREE.PerspectiveCamera(
          75,
          overlay.value.clientWidth / overlay.value.clientHeight,
          0.1,
          1000
      );
      camera.value.position.z = 1.7;

      const light = new THREE.DirectionalLight(0xffffff, 1);
      light.position.set(0, 2, 2);
      scene.value.add(light);

      const ambientLight = new THREE.AmbientLight(0xffffff, 0.5);
      scene.value.add(ambientLight);
    };

    const startVideo = async () => {
      try {
        video.value.srcObject = await navigator.mediaDevices.getUserMedia({video: {}});
      } catch (err) {
        console.error('Error accessing webcam:', err);
      }
    };

    const detectFace = async () => {
      const displaySize = {
        width: video.value.videoWidth,
        height: video.value.videoHeight,
      };

      faceapi.matchDimensions(overlay.value, displaySize);

      detectionInterval = setInterval(async () => {
        const detections = await faceapi
            .detectAllFaces(video.value, new faceapi.TinyFaceDetectorOptions())
            .withFaceLandmarks();

        if (detections.length > 0) {
          const landmarks = detections[0].landmarks;
          positionGlasses(landmarks);
        }

        renderer.value.render(scene.value, camera.value);
      }, 100);
    };

    const positionGlasses = (landmarks) => {
      const leftEye = landmarks.getLeftEye();
      const rightEye = landmarks.getRightEye();

      // محاسبه مرکز چشم‌ها
      const eyeCenter = {
        x: (leftEye[0].x + rightEye[3].x) / 2,
        y: (leftEye[0].y + rightEye[3].y) / 2,
      };

      // فاصله بین دو چشم
      const eyeDistance = rightEye[3].x - leftEye[0].x;

      // تبدیل مختصات چشم‌ها به سیستم مختصات سه‌بعدی
      const normalizedX = ((eyeCenter.x / video.value.videoWidth) * 2 );
      const normalizedY = -((eyeCenter.y / video.value.videoHeight) * 2 );

      // تنظیم افست دقیق
      const xOffset = -4.56; // جابه‌جایی به چپ
      const yOffset = 0; // جابه‌جایی به بالا

      glassesModel.value.position.set(
          normalizedX + xOffset,
          normalizedY + yOffset,
          -3
      );

      // تنظیم مقیاس
      const scaleFactor = eyeDistance / 80; // تنظیم دقیق‌تر
      glassesModel.value.scale.set(scaleFactor, scaleFactor, scaleFactor);

      // تنظیم چرخش برای صاف شدن
      glassesModel.value.rotation.set(0, 0.6, 0);
    };

    onMounted(async () => {
      await loadFaceApiModels();
      setupThreeJS();
      loadGlassesModel(scene);
      await startVideo();

      video.value.addEventListener('play', detectFace);
    });

    onBeforeUnmount(() => {
      clearInterval(detectionInterval);
    });

    return {video, overlay};
  },
};
</script>

<style scoped>
.video-stream {
  position: relative;
}

video {
  width: 100%;
  height: auto;
  display: block;
}

canvas {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}
</style>
