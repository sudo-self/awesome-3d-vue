# <a href="awesome-3s-vue.vercel.app">awesome-3D-vue</a>
<img width="1440" alt="og" src="https://github.com/user-attachments/assets/eedabdd2-a795-4119-8a35-e84338511d39" />

<center>
  <a href="https://deploy.workers.cloudflare.com/?url=https://github.com/sudo-self/awesome-3d-vue.git">
    <img src="https://deploy.workers.cloudflare.com/button" alt="Deploy to Cloudflare Workers" />
  </a>
</center>
                

## App.vue

```
<template>
  <div id="app">
    <div ref="threeContainer" class="three-container"></div>
    <div v-show="controlsVisible" class="controls">

      <label for="hdrToggle">HDR</label>
      <input type="checkbox" id="hdrToggle" v-model="useHDR" />

      <label for="bgColor">Background</label>
      <input type="color" id="bgColor" v-model="bgColor" />

      <label for="cameraFov">FOV</label>
      <input
        type="range"
        id="cameraFov"
        min="10"
        max="100"
        step="1"
        v-model="cameraFov"
      />
      <span>{{ cameraFov }}</span>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import * as THREE from 'three';
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader';
import { RGBELoader } from 'three/examples/jsm/loaders/RGBELoader';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';

const threeContainer = ref(null);
const controlsVisible = ref(true);
const ambientLightIntensity = ref(2.6);
const directionalLightIntensity = ref(1.4);
const useHDR = ref(true);
const bgColor = ref('#000000');
const cameraFov = ref(75);

onMounted(() => {
  const scene = new THREE.Scene();
  const camera = new THREE.PerspectiveCamera(cameraFov.value, window.innerWidth / window.innerHeight, 0.1, 1000);
  const renderer = new THREE.WebGLRenderer();

  renderer.setSize(window.innerWidth, window.innerHeight);
  threeContainer.value.appendChild(renderer.domElement);

  const ambientLight = new THREE.AmbientLight(0xffffff, ambientLightIntensity.value);
  scene.add(ambientLight);

  const directionalLight = new THREE.DirectionalLight(0xffffff, directionalLightIntensity.value);
  directionalLight.position.set(1, 1, 1).normalize();
  scene.add(directionalLight);


  const hdrLoader = new RGBELoader();
  let hdrTexture = null;

  hdrLoader.load('model.hdr', (texture) => {
    texture.mapping = THREE.EquirectangularRefractionMapping;
    hdrTexture = texture;
    if (useHDR.value) {
      scene.background = texture;
      scene.environment = texture;
    }
  });


  const loader = new GLTFLoader();
  loader.load('/model.gltf', (gltf) => {
    const model = gltf.scene;
    model.scale.set(1.7, 1.7, 1.7);
    model.position.set(0, -5, 0);
    scene.add(model);
  });

  camera.position.z = -5;
  camera.up.set(0, 2, 0);

  const controls = new OrbitControls(camera, renderer.domElement);
  controls.enableDamping = true;
  controls.dampingFactor = 0.25;
  controls.screenSpacePanning = false;

  const animate = () => {
    requestAnimationFrame(animate);

    if (hdrTexture) {
      if (useHDR.value) {
        scene.background = hdrTexture;
        scene.environment = hdrTexture;
      } else {
        scene.background = new THREE.Color(bgColor.value);
        scene.environment = null;
      }
    } else {
      scene.background = new THREE.Color(bgColor.value);
    }

    camera.fov = cameraFov.value;
    camera.updateProjectionMatrix();

 
    controls.update();
    renderer.render(scene, camera);
  };

  animate();
});

const toggleControls = () => {
  controlsVisible.value = !controlsVisible.value;
};
</script>

<style scoped>
.three-container {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: #000;
}

.controls {
  position: absolute;
  top: 10px;
  right: calc(50% - 20px);
  display: flex;
  background-color: rgba(0, 0, 0, 0.7);
  color: white;
  z-index: 10;
  overflow-y: auto;
  max-height: calc(100% - 60px);
}

.controls label,
.controls span {
  display: inline-block;
  margin-top: 8px;
}

.controls input[type='range'] {
  width: 100%;
  margin-top: 5px;
}

.controls input[type='color'] {
  width: 100%;
  margin-top: 5px;
}
</style>

```

