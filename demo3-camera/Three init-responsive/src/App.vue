<template></template>

<script setup>
import * as THREE from "three";
import Stats from "three/examples/jsm/libs/stats.module.js";
import { FBXLoader } from "three/examples/jsm/loaders/FBXLoader.js";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js";
import { Octree } from "three/examples/jsm/math/Octree.js";
import { OctreeHelper } from "three/examples/jsm/helpers/OctreeHelper.js";
import { Capsule } from "three/examples/jsm/math/Capsule.js";
import { GUI } from "three/examples/jsm/libs/lil-gui.module.min.js";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import { reactive, onMounted, ref, watch } from "vue";
import * as dat from "dat.gui";
import CameraControls from "camera-controls";
CameraControls.install({ THREE: THREE });
//全局
let scene,
  renderer,
  camera,
  fov,
  near,
  far,
  cameracontrols,
  width,
  height,
  raycaster,
  clock,
  cube,
  light;
let renderRequested = false;
scene = new THREE.Scene();
renderer = new THREE.WebGLRenderer({
  antialias: true,
});
raycaster = new THREE.Raycaster();
const gridHelper = new THREE.GridHelper(50, 50);
gridHelper.position.y = -1;
scene.add(gridHelper);
clock = new THREE.Clock();
// const mesh = new THREE.Mesh(
//   new THREE.BoxGeometry(1, 1, 1),
//   new THREE.MeshBasicMaterial({ color: 0xff0000, wireframe: true })
// );
// mesh.onBeforeRender = function () {
//   this.rotation.x += 0.01;
// };
// scene.add(mesh);
const MAX_POINTS = 500;

// geometry
const geometry = new THREE.BufferGeometry();

// attributes
let positions = new Float32Array( MAX_POINTS * 3 ); // 3 vertices per point
geometry.setAttribute( 'position', new THREE.BufferAttribute( positions, 3 ) );

// draw range
const drawCount = 2; // draw the first 2 points, only
geometry.setDrawRange( 0, drawCount );

// material
const material = new THREE.LineBasicMaterial( { color: 0xff0000 } );

// line
const line = new THREE.Line( geometry, material );
//  positions = line.geometry.attributes.position.array;

// let x, y, z, index;
// x = y = z = index = 0;

// for ( let i = 0, l = MAX_POINTS; i < l; i ++ ) {

//     positions[ index ++ ] = x;
//     positions[ index ++ ] = y;
//     positions[ index ++ ] = z;

//     x += ( Math.random() - 0.5 ) * 30;
//     y += ( Math.random() - 0.5 ) * 30;
//     z += ( Math.random() - 0.5 ) * 30;

// }
console.log(line)
scene.add( line );
onMounted(() => {
  width = window.innerWidth;
  height = window.innerHeight;
  fov = 60;
  near = 0.01;
  far = 10000;
  camera = new THREE.PerspectiveCamera(fov, width / height, near, far);
  camera.position.set(0, 0, 5);
  renderer.setSize(width, height);
  renderer.shadowMap.enabled = true;
  renderer.render(scene, camera);
  document.body.appendChild(renderer.domElement);

  cameracontrols = new CameraControls(camera, renderer.domElement);
  // cameracontrols.fitToBox(line)
  let animate = function () {
    renderRequested = false;
    cameracontrols.update(clock.getDelta());
    requestAnimationFrame(animate);
    renderer.render(scene, camera);
  };

  animate();
  renderer.domElement.addEventListener("click", mouseClick, false);
});
function mouseClick(e) {
  e.preventDefault();
  let mouse = new THREE.Vector2();
  // 通过鼠标点击的位置计算出raycaster所需要的点的位置，以屏幕中心为原点，值的范围为-1到1.
  mouse.x =
    ((e.clientX - renderer.domElement.getBoundingClientRect().left) / width) *
      2 -
    1;
  mouse.y =
    -((e.clientY - renderer.domElement.getBoundingClientRect().top) / height) *
      2 +
    1;
  raycaster.setFromCamera(mouse, camera);
  const intersects = raycaster.intersectObjects(scene.children);
  console.log(intersects);
}
window.addEventListener("resize", () => {
  // 更新渲染
  renderer.setSize(window.innerWidth, window.innerHeight);
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
  // 更新相机
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
});

</script>

<style>
* {
  margin: 0;
  padding: 0;
  overflow: hidden;
}
</style>
