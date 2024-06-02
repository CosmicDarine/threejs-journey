<template>
  <canvas class="webgl" />
</template>

<script setup lang="ts">
import { onMounted, ref, shallowRef } from "vue";
import * as THREE from "three";
import { OrbitControls } from "three/addons/controls/OrbitControls.js";
import GUI from 'lil-gui';

const gui = new GUI();
const debugObjet = ref<any>({ 
  color: 0xff0000});

const sizes = {
  width: window.innerWidth,
  height: window.innerHeight,
};

const counter = ref<number>(0);
const scene = shallowRef(new THREE.Scene());

// const positionsArray = new Float32Array([
//   0,0,0,
//   0,1,0,
//   1,0,0
// ]);

// const positionsAttribute = new THREE.BufferAttribute(positionsArray, 3);
// geometry.value.setAttribute('position', positionsAttribute);
// const geometry = shallowRef(new THREE.BufferGeometry());
// const count = 50;
// const positionsArray = new Float32Array(count * 3 * 3);
// const sphere = new THREE.SphereGeometry(1);

const geometry = shallowRef(new THREE.BoxGeometry(1, 1, 1, 4,4,4));
const material = shallowRef(new THREE.MeshBasicMaterial({ 
  color: 0xff0000,
  // wireframe: true, 
}));
const mesh = shallowRef(new THREE.Mesh(geometry.value, material.value));

const camera = shallowRef(
  new THREE.PerspectiveCamera(75, sizes.width / sizes.height)
);
const canvas = shallowRef<HTMLCanvasElement | null>(null);
const renderer = shallowRef<THREE.WebGLRenderer | null>(null);
const clock = shallowRef<THREE.Clock | null>(null);
const controls = shallowRef<OrbitControls | null>(null);

window.addEventListener('resize', () => {
  sizes.width = window.innerWidth;
  sizes.height = window.innerHeight;

  //update camera and aspect
  camera.value.aspect = sizes.width / sizes.height;
  camera.value.updateProjectionMatrix();

  //update renderer
  renderer.value?.setSize(sizes.width, sizes.height);
});

window.addEventListener('dblclick', () => {

});

onMounted(() => {
  try {
    clock.value = new THREE.Clock();
    canvas.value = document.querySelector("canvas.webgl") as HTMLCanvasElement;
    
    if (!canvas.value) {
      console.error("Canvas not found");
      return;
    }
    
    renderer.value = new THREE.WebGLRenderer({
      canvas: canvas.value,
    });
    renderer.value.setSize(sizes.width, sizes.height);
    renderer.value.setPixelRatio(Math.min(window.devicePixelRatio,2))
    scene.value.add(mesh.value);
    scene.value.add(camera.value);

    gui.add(mesh.value.position, 'y').min(-3).max(3).step(0.01).name('elevation (y)')
    gui.add(material.value, 'wireframe');
    gui.add(mesh.value, 'visible')
    gui.addColor(debugObjet.value, 'color').onChange(() => {
      material.value.color.set(debugObjet.value.color);
    })
    debugObjet.value.subdivision = 2;
    gui.add(debugObjet.value, 'subdivision')
    .min(1)
    .max(20)
    .step(1)
    .onFinishChange(() => {
      mesh.value.geometry.dispose();
      mesh.value.geometry = new THREE.BoxGeometry(
        1,1,1,
        debugObjet.value.subdivision,debugObjet.value.subdivision,debugObjet.value.subdivision
      );
    });
    camera.value.lookAt(mesh.value.position);
    camera.value.position.z = 3;
    controls.value = new OrbitControls(camera.value, canvas.value);
    controls.value.enableDamping = true;
    // controls.value.enabled = false;
    controls.value.update();

    animate(); 
  } catch (error) {
    console.error("An error occurred in the mounted hook", error);
  }
});

const animate = () => {
      requestAnimationFrame(animate);
      counter.value++;
      
      // Update controls
      controls.value?.update();
      
      // Render the scene
      renderer.value?.render(scene.value, camera.value);
    };
</script>

<style scoped></style>
