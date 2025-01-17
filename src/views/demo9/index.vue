<template>
  <div ref="threeDom" class="container"></div>
</template>

<script lang="ts" setup>
import * as THREE from "three";
import { ref, onMounted } from "vue";
// 导入轨道控制器
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
// 导入动画库
import gsap from "gsap";
// 导入dat.gui
import * as dat from "dat.gui";
import particles1Img from "@/assets/textures/particles/1.png";

const threeDom: any = ref(null);

onMounted(() => {
  // 1、创建场景
  const scene = new THREE.Scene();

  // 2、创建相机
  const camera = new THREE.PerspectiveCamera(
    75,
    window.innerWidth / window.innerHeight,
    0.1,
    30
  );

  // 设置相机位置
  camera.position.set(0, 0, 40);
  scene.add(camera);

  function createPoints(url: any, size = 0.5, color = [0.5, 0.5, 0.5]) {
    const particlesGeometry = new THREE.BufferGeometry();
    const count = 1000;

    // 设置缓冲区数组
    const positions = new Float32Array(count * 3);
    // 设置粒子顶点颜色
    const colors = new Float32Array(count * 3);
    // 设置顶点
    for (let i = 0; i < count * 3; i++) {
      positions[i] = (Math.random() - 0.5) * 100;
      if (i % 3 === 0) {
        colors[i] = color[0] / 255;
      } else if (i % 3 === 1) {
        colors[i] = color[1] / 255;
      } else {
        colors[i] = color[2] / 255;
      }
    }
    particlesGeometry.setAttribute(
      "position",
      new THREE.BufferAttribute(positions, 3)
    );
    particlesGeometry.setAttribute(
      "color",
      new THREE.BufferAttribute(colors, 3)
    );

    // 设置点材质
    const pointsMaterial = new THREE.PointsMaterial();
    pointsMaterial.size = size;
    pointsMaterial.color.set(0xffffff);
    // 相机深度而衰减
    pointsMaterial.sizeAttenuation = true;

    // 载入纹理
    const textureLoader = new THREE.TextureLoader();

    const obj: any = {
      1: particles1Img,
    };

    const texture = textureLoader.load(obj[url]);
    // 设置点材质纹理
    pointsMaterial.map = texture;
    pointsMaterial.alphaMap = texture;
    pointsMaterial.transparent = true;
    pointsMaterial.depthWrite = false;
    pointsMaterial.blending = THREE.AdditiveBlending;
    // 设置启动顶点颜色
    pointsMaterial.vertexColors = true;

    const points = new THREE.Points(particlesGeometry, pointsMaterial);

    scene.add(points);
    return points;
  }

  const points = createPoints("1", 0.5, [0, 163, 255]);
  console.log(points);
  const points2 = createPoints("1", 0.5, [48, 70, 198]);
  const points3 = createPoints("1", 0.5, [85, 11, 142]);

  // 初始化渲染器
  const renderer = new THREE.WebGLRenderer();
  // 设置渲染的尺寸大小
  renderer.setSize(window.innerWidth, window.innerHeight);
  // 开启场景中的阴影贴图
  renderer.shadowMap.enabled = true;
  renderer.physicallyCorrectLights = true;

  // console.log(renderer);
  // 将webgl渲染的canvas内容添加到body
  threeDom.value.appendChild(renderer.domElement);

  // // 使用渲染器，通过相机将场景渲染进来
  // renderer.render(scene, camera);

  // 创建轨道控制器
  const controls = new OrbitControls(camera, renderer.domElement);
  // 设置控制器阻尼，让控制器更有真实效果,必须在动画循环里调用.update()。
  controls.enableDamping = true;

  // 添加坐标轴辅助器
  const axesHelper = new THREE.AxesHelper(5);
  scene.add(axesHelper);
  // 设置时钟
  const clock = new THREE.Clock();
  console.log(clock);

  function render() {
    let time = clock.getElapsedTime();
    points.rotation.x = time * 0.05;
    points.rotation.y = time * -0.05;
    if (Math.floor(time) % 10 <= 3 && points.material.opacity > 0) {
      points.material.opacity -= 1 / 180;
      if (points.material.opacity < 0) points.material.opacity = 0;
    } else if (Math.floor(time) % 10 > 3 && points.material.opacity < 1) {
      points.material.opacity += 1 / 180;
      if (points.material.opacity > 1) points.material.opacity = 1;
    }
    points2.rotation.x = time * 0.05;
    points2.rotation.y = time * -0.05;
    // points2.material.opacity = 0;
    points3.rotation.x = time * 0.05;
    points3.rotation.y = time * -0.05;
    // points3.material.opacity = 0;
    controls.update();
    renderer.render(scene, camera);
    //   渲染下一帧的时候就会调用render函数
    requestAnimationFrame(render);
  }

  render();

  // 监听画面变化，更新渲染画面
  window.addEventListener("resize", () => {
    // 更新摄像头
    camera.aspect = window.innerWidth / window.innerHeight;
    //   更新摄像机的投影矩阵
    camera.updateProjectionMatrix();

    //   更新渲染器
    renderer.setSize(window.innerWidth, window.innerHeight);
    //   设置渲染器的像素比
    renderer.setPixelRatio(window.devicePixelRatio);
  });
});
</script>

<style lang="scss">
.container {
  height: 100%;
}
</style>
