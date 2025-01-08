// src/components/VideoStream.vue
<template>
  <div class="video-stream">
    <!-- Video element for displaying the webcam stream -->
    <video id="video" ref="video" autoplay muted></video>
    <!-- Canvas overlay for drawing detected face landmarks or glasses -->
    <canvas id="overlay" ref="overlay"></canvas>
  </div>
</template>

<script>
import * as faceapi from 'face-api.js'; // Import face-api.js library
import * as THREE from "three"

export default {
  name: 'TryOnGlasses',
  data() {
    return {
      video: null, // Reference to the video element
      canvas: null, // Reference to the overlay canvas
      modelsLoaded: false, // Tracks whether face-api.js models are loaded
      detectionInterval: null, // Interval ID for face detection updates
    };
  },
  methods: {
    async loadModels() {
      // Load FaceAPI models for face detection and facial landmarks
      await faceapi.nets.tinyFaceDetector.loadFromUri('/models/faceApi'); // Load tiny face detector model
      await faceapi.nets.faceLandmark68Net.loadFromUri('/models/faceApi'); // Load face landmark model
      this.modelsLoaded = true; // Mark models as loaded
      console.log('Models loaded');
    },
    async startVideo() {
      // Access the video element by its ID
      // this.video = document.getElementById('video');
      this.video = this.$refs.video;
      try {
        // Request access to the user's webcam
        // Attach the webcam stream to the video element
        this.video.srcObject = await navigator.mediaDevices.getUserMedia({video: {}});
      } catch (err) {
        // Log an error if webcam access fails
        console.error('Error accessing webcam:', err);
      }
    },
    async detectFace() {
      // Ensure models are loaded before running detection
      if (!this.modelsLoaded) {
        console.error('Models not loaded');
        return;
      }

      // Get the canvas element and its 2D drawing context
      // const overlayCanvas = document.getElementById('overlay');
      const overlayCanvas = this.$refs.overlay;
      const ctx = overlayCanvas.getContext('2d');

      // Match the canvas size to the video element's dimensions
      overlayCanvas.width = this.video.videoWidth;
      overlayCanvas.height = this.video.videoHeight;

      const displaySize = {
        width: this.video.videoWidth,
        height: this.video.videoHeight,
      };

      // Adjust detection results to match the display size
      faceapi.matchDimensions(overlayCanvas, displaySize);

      // Set up periodic face detection
      this.detectionInterval = setInterval(async () => {
        // Detect faces and landmarks using FaceAPI
        const detections = await faceapi
            .detectAllFaces(this.video, new faceapi.TinyFaceDetectorOptions())
            .withFaceLandmarks();

        // Clear the canvas before drawing new detections
        ctx.clearRect(0, 0, overlayCanvas.width, overlayCanvas.height);

        if (detections.length > 0) {
          const landmarks = detections[0].landmarks; // Get landmarks of the first detected face
          this.drawGlasses(ctx, landmarks); // Draw glasses overlay on the detected face
        }
      }, 100); // Run detection every 100ms
    },
    drawGlasses(ctx, landmarks) {
      // Get landmarks for the left and right eyes
      const leftEye = landmarks.getLeftEye();
      const rightEye = landmarks.getRightEye();

      // Calculate the width and height of the glasses based on eye position
      const eyeWidth = rightEye[3].x - leftEye[0].x; // Distance between outer corners of the eyes
      const eyeHeight = (leftEye[3].y - leftEye[0].y) * 2; // Approximate height of glasses

      // Position the glasses slightly above and around the eyes
      const glassesX = leftEye[0].x - eyeWidth * 0.2; // X-position offset for glasses
      const glassesY = leftEye[0].y - eyeHeight * 0.8; // Y-position offset for glasses

      // Draw the glasses as a filled rectangle
      ctx.fillStyle = 'rgba(0, 0, 0, 0.7)'; // Semi-transparent black fill for glasses
      ctx.fillRect(glassesX, glassesY, eyeWidth * 1.4, eyeHeight); // Glasses dimensions

      // Add a border around the glasses
      ctx.strokeStyle = 'black'; // Black border color
      ctx.lineWidth = 2; // Border thickness
      ctx.strokeRect(glassesX, glassesY, eyeWidth * 1.4, eyeHeight); // Border dimensions
    },
  },
  mounted() {
    // Load models and start video stream when the component is mounted
    this.loadModels(); // Load FaceAPI models
    this.startVideo(); // Start webcam video
    this.video.addEventListener('play', this.detectFace); // Begin face detection when video starts playing
  },
  beforeUnmount() {
    // Clear detection interval to prevent memory leaks
    clearInterval(this.detectionInterval);
  },
};
</script>

<style scoped>
.video-stream {
  position: relative; /* Ensure video and canvas are overlaid */
}
video {
  width: 100%;
  height: auto; /* Maintain video aspect ratio */
  display: block; /* Remove any inline spacing */
}
canvas {
  position: absolute; /* Overlay the canvas on the video */
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}
</style>
