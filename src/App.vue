<template>
  <div class="player">
    <div>
      <button @click="$refs.video.play()">播放</button>
    </div>
    <video
      preload
      ref="video"
      controls
      loop
      style="width: 100%; visibility: hidden; position: absolute"
      src="./assets/360° air show - PC-7 TEAM @ FIS Ski World Cup 2016 - St. Moritz.mp4"
    ></video>

    <canvas
      style="width: 80%; height: 823px"
      width="1920"
      height="823"
      ref="canvas"
    ></canvas>
  </div>
</template>
<script>
import * as THREE from "three/build/three.module";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
export default {
  name: 'App',
  data() {
    return {
      isPlay: false,
      canPlay: false,
    };
  },
  methods: {
    initOffScreenCanvas() {
      this.videoCanvas = new OffscreenCanvas();
    },
    initVideoTexture() {
      this.videoTexture = new THREE.VideoTexture(this.$refs.video);
      this.videoTexture.needsUpdate = true;
      this.videoTexture.updateMatrix();
    },
    initRenderer() {
      this.renderer = new THREE.WebGLRenderer({
        canvas: this.$refs.canvas,
        antialias: true,
      });
    },
    initScene() {
      this.scene = new THREE.Scene();
    },
    initMesh() {
      this.geometry = new THREE.SphereGeometry(100, 32, 16);
      this.material = new THREE.ShaderMaterial({
        wireframe: false,
        side: THREE.DoubleSide,
        map: this.videoTexture,
        uniforms: {
          tex_0: new THREE.Uniform(this.videoTexture),
        },
        vertexShader: require("@/components/v.js").default,
        fragmentShader: require("@/components/f.js").default,
      });
      this.mesh = new THREE.Mesh(this.geometry, this.material);
    },
    initCamera() {
      this.camera = new THREE.PerspectiveCamera(45, 1024 / 768, 1, 1000);
      this.camera.position.z = 30;
      this.controls = new OrbitControls(this.camera, this.renderer.domElement);
      this.controls.maxDistance = 100;

      this.controls.update();
      // const helper = new THREE.CameraHelper(this.camera);
      // this.scene.add(helper);
      this.scene.add(this.camera);
    },
    addMeshToScene() {
      this.scene.add(this.mesh);
    },
    update() {
      this.renderer.render(this.scene, this.camera);

      requestAnimationFrame(this.update);
    },
  },
  mounted() {
    this.initRenderer();
    this.initScene();
    this.initVideoTexture();
    this.initMesh();
    this.initCamera();
    this.addMeshToScene();
    this.update();
    window.addEventListener('deviceorientation', function(event) {
  console.log('alpha: ' + event.alpha + ', beta: ' + event.beta + ', gamma: ' + event.gamma);
});
  },
};
</script>