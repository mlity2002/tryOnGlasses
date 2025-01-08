<template>
<!--  <ARScene model-url="models/glasses.glb" />-->
<!--  <face-api />-->
<!--  <TryOnGlasses />-->
  <TryOnGlass />
</template>

<script>
import ARScene from "@/components/ARScene.vue";
import FaceApi from "@/components/faceApi.vue";
import TryOnGlasses from "@/components/TryOnGlasses.vue";
import TryOnGlass from "@/components/TryOnGlass.vue";

export default {
  components: {
    TryOnGlass,
    TryOnGlasses,
    FaceApi,
    ARScene,
  },
};
</script>

<style></style>
