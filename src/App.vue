<template>
  <div ref="container" class="galaxy-container"></div>
</template>

<script setup lang="ts">
import {GUI} from "dat.gui";
import * as THREE from "three";
import {OrbitControls} from "three-stdlib";
import {onMounted, onUnmounted, ref} from "vue";

const container = ref<HTMLDivElement | null>(null); // HTML 요소 참조를 위한 ref 설정

// 은하 생성에 필요한 매개변수 설정
const parameters = {
  count: 100000, // 은하의 입자 수
  size: 0.001, // 입자의 크기
  radius: 10, // 은하의 반경
  branches: 3, // 은하의 팔 수
  spin: 1, // 은하의 회전량
  randomness: 0.3, // 입자의 무작위성
  randomnessPower: 3, // 입자의 무작위성 강도
  insideColor: "#ff5334", // 은하 내부의 색상
  outsideColor: "#05335c", // 은하 외부의 색상
};

let geometry: THREE.BufferGeometry | null = null; // BufferGeometry 객체
let material: THREE.PointsMaterial | null = null; // PointsMaterial 객체
let points: THREE.Points | null = null; // Points 객체
let scene: THREE.Scene | null = null; // Three.js 장면(Scene) 객체

// 은하 생성 함수
const generateGalaxy = () => {
  if (!scene) return;

  // 기존 은하 제거
  if (points !== null) {
    geometry?.dispose();
    material?.dispose();
    scene.remove(points);
  }

  // 지오메트리 설정
  geometry = new THREE.BufferGeometry();

  const positions = new Float32Array(parameters.count * 3); // 입자 위치 배열
  const colors = new Float32Array(parameters.count * 3); // 입자 색상 배열

  const colorInside = new THREE.Color(parameters.insideColor); // 내부 색상
  const colorOutside = new THREE.Color(parameters.outsideColor); // 외부 색상

  // 입자 데이터 생성
  for (let i = 0; i < parameters.count; i++) {
    const i3 = i * 3;

    const radius = Math.random() * parameters.radius; // 무작위 반경
    const spinAngle = radius * parameters.spin; // 스핀 각도
    const branchAngle =
      ((i % parameters.branches) / parameters.branches) * Math.PI * 2; // 팔 각도

    const randomX = (Math.random() - 0.5) * parameters.randomness * radius; // X축 무작위성
    const randomY = (Math.random() - 0.5) * parameters.randomness * radius; // Y축 무작위성
    const randomZ = (Math.random() - 0.5) * parameters.randomness * radius; // Z축 무작위성

    positions[i3] = Math.cos(branchAngle + spinAngle) * radius + randomX;
    positions[i3 + 1] = randomY;
    positions[i3 + 2] = Math.sin(branchAngle + spinAngle) * radius + randomZ;

    // 색상 혼합
    const mixedColor = colorInside.clone();
    mixedColor.lerp(colorOutside, radius / parameters.radius);

    colors[i3] = mixedColor.r;
    colors[i3 + 1] = mixedColor.g;
    colors[i3 + 2] = mixedColor.b;
  }

  geometry.setAttribute("position", new THREE.BufferAttribute(positions, 3)); // 위치 속성 설정
  geometry.setAttribute("color", new THREE.BufferAttribute(colors, 3)); // 색상 속성 설정

  // 재질(Material) 설정
  material = new THREE.PointsMaterial({
    size: parameters.size,
    sizeAttenuation: true,
    depthWrite: false,
    blending: THREE.AdditiveBlending,
    vertexColors: true,
  });

  // 포인트(Point) 객체 생성 및 장면(Scene)에 추가
  points = new THREE.Points(geometry, material);
  scene.add(points);
};

onMounted(() => {
  if (!container.value) return;

  // 장면(Scene) 생성
  scene = new THREE.Scene();

  // 카메라 생성
  const camera = new THREE.PerspectiveCamera(
    75, // 시야각
    window.innerWidth / window.innerHeight, // 종횡비
    0.001, // 근거리
    100000 // 원거리
  );
  camera.position.y = 5; // 카메라 위치 설정
  camera.position.z = 5;

  // 렌더러(Renderer) 생성 및 설정
  const renderer = new THREE.WebGLRenderer({antialias: true});
  renderer.setSize(window.innerWidth, window.innerHeight);
  container.value.appendChild(renderer.domElement); // 렌더러 DOM 요소 추가

  // OrbitControls 설정
  const controls = new OrbitControls(camera, renderer.domElement);

  // 애니메이션 함수
  const animate = () => {
    requestAnimationFrame(animate);
    if (points) {
      points.rotation.y += 0.001; // 은하 회전 속도 조정
    }
    controls.update();
    renderer.render(scene, camera);
  };

  generateGalaxy(); // 은하 생성

  // dat.GUI 설정
  const gui = new GUI();
  gui
    .add(parameters, "count")
    .min(100)
    .max(1000000)
    .step(100)
    .onFinishChange(generateGalaxy); // 은하 입자 수 변경 시 갱신
  gui
    .add(parameters, "size")
    .min(0.001)
    .max(0.1)
    .step(0.001)
    .onFinishChange(generateGalaxy); // 은하 입자 크기 변경 시 갱신
  gui
    .add(parameters, "radius")
    .min(0.01)
    .max(20)
    .step(0.01)
    .onFinishChange(generateGalaxy); // 은하 반경 변경 시 갱신
  gui
    .add(parameters, "branches")
    .min(2)
    .max(20)
    .step(1)
    .onFinishChange(generateGalaxy); // 은하 팔 수 변경 시 갱신
  gui
    .add(parameters, "spin")
    .min(-5)
    .max(5)
    .step(0.001)
    .onFinishChange(generateGalaxy); // 은하 회전량 변경 시 갱신
  gui
    .add(parameters, "randomness")
    .min(0)
    .max(2)
    .step(0.001)
    .onFinishChange(generateGalaxy); // 입자 무작위성 변경 시 갱신
  gui
    .add(parameters, "randomnessPower")
    .min(1)
    .max(10)
    .step(0.001)
    .onFinishChange(generateGalaxy); // 입자 무작위성 강도 변경 시 갱신
  gui.addColor(parameters, "insideColor").onFinishChange(generateGalaxy); // 내부 색상 변경 시 갱신
  gui.addColor(parameters, "outsideColor").onFinishChange(generateGalaxy); // 외부 색상 변경 시 갱신

  animate(); // 애니메이션 시작

  // 윈도우 크기 조정 이벤트 처리
  const onResize = () => {
    camera.aspect = window.innerWidth / window.innerHeight;
    camera.updateProjectionMatrix();
    renderer.setSize(window.innerWidth, window.innerHeight);
  };

  window.addEventListener("resize", onResize);

  // 컴포넌트 언마운트 시 리소스 정리
  onUnmounted(() => {
    window.removeEventListener("resize", onResize);
    gui.destroy();
    renderer.dispose();
    controls.dispose();
  });
});
</script>

<style scoped>
.galaxy-container {
  width: 100%;
  height: 100vh;
  overflow: hidden;
}
</style>
