<template>
  <div id="container">
    <canvas id="planet" ref="canvas"></canvas>
  </div>
</template>

<script>
import * as THREE from 'three';
import * as Controls from 'three-orbit-controls';

const OrbitControls = Controls(THREE);

export default {
  name: 'planet',
  data() {
    return {
      loader: new THREE.TextureLoader(),
      scene: new THREE.Scene(),
      light: new THREE.SpotLight(0xFFFFFF, 1, 0, Math.PI / 2, 1),
      renderer: null,
    };
  },
  mounted() {
    const width = window.innerWidth;
    const height = window.innerHeight;
    // Earth params
    const radius = 0.5;
    const segments = 32;
    const rotation = 6;

    const scene = new THREE.Scene();

    const camera = new THREE.PerspectiveCamera(45, width / height, 0.01, 1000);
    camera.position.z = 1.5;

    const renderer = new THREE.WebGLRenderer({ antialias: true, canvas: this.$refs.canvas });
    renderer.setSize(width, height);

    scene.add(new THREE.AmbientLight(0x333333, 0.375));

    const light = new THREE.DirectionalLight(0xffffff, 1);
    light.position.set(5, 3, 5);
    scene.add(light);

    const sphere = this.createSphere(radius, segments);
    sphere.rotation.y = rotation;
    scene.add(sphere);

    const clouds = this.createClouds(radius, segments);
    clouds.rotation.y = rotation;
    scene.add(clouds);

    const stars = this.createStars(90, 64);
    scene.add(stars);

    const controls = new OrbitControls(camera);
    controls.maxDistance = 100;
    controls.minDistance = 0.8;

    function render() {
      controls.update();
      sphere.rotation.y += 0.0005;
      clouds.rotation.y += 0.0004;
      requestAnimationFrame(render);
      renderer.render(scene, camera);
    }

    render();
  },
  methods: {
    earthMap() {
      return require('../assets/2_no_clouds_4k.jpg'); // eslint-disable-line global-require
    },
    earthBumpMap() {
      return require('../assets/elev_bump_4k.jpg'); // eslint-disable-line global-require
    },
    earthSpecularMap() {
      return require('../assets/water_4k.png'); // eslint-disable-line global-require
    },
    cloudsMap() {
      return require('../assets/fair_clouds_4k.png'); // eslint-disable-line global-require
    },
    starsMap() {
      return require('../assets/galaxy_starfield.png'); // eslint-disable-line global-require
    },
    createSphere(radius, segments) {
      return new THREE.Mesh(
        new THREE.SphereGeometry(radius, segments, segments),
        new THREE.MeshPhongMaterial({
          map: this.loader.load(this.earthMap()),
          bumpMap: this.loader.load(this.earthBumpMap()),
          bumpScale: 0.01,
          specularMap: this.loader.load(this.earthSpecularMap()),
          specular: new THREE.Color('grey'),
          shininess: 10,
        }),
      );
    },
    createClouds(radius, segments) {
      return new THREE.Mesh(
        new THREE.SphereGeometry(radius + 0.003, segments, segments),
        new THREE.MeshPhongMaterial({
          map: this.loader.load(this.cloudsMap()),
          transparent: true,
        }),
      );
    },
    createStars(radius, segments) {
      return new THREE.Mesh(
        new THREE.SphereGeometry(radius, segments, segments),
        new THREE.MeshBasicMaterial({
          map: this.loader.load(this.starsMap()),
          side: THREE.BackSide,
        }),
      );
    },
    createLights(radius, segments) {
      return new THREE.Mesh(
        new THREE.SphereGeometry(radius, segments, segments, 180, Math.PI * 1, 0, Math.PI),
        new THREE.MeshBasicMaterial({
          map: this.loader.load(this.starsMap()),
          side: THREE.BackSide,
        }),
      );
    },
  },
};
</script>
