<template>
</template>

<script lang="ts">
import {
  Component,
  Inject,
  Model,
  Prop,
  Vue,
  Provide,
  Watch,
} from "nuxt-property-decorator"

import * as THREE from 'three';
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';
import { RoomEnvironment } from 'three/examples/jsm/environments/RoomEnvironment.js';

@Component
export default  class Index extends Vue {
  room: any= undefined;
  scene: any= undefined;
  camera: any = undefined;
  renderer: any = undefined;
  loaderModel: any= undefined;
  controls: any = undefined;

  initObject() {
    this.scene = new THREE.Scene();
		this.camera = new THREE.PerspectiveCamera( 75, window.innerWidth / window.innerHeight, 0.1, 1000 );
    this.loaderModel = new GLTFLoader();
    this.renderer = new THREE.WebGLRenderer({ antialias: true });
    this.controls = new OrbitControls(this.camera, this.renderer.domElement);
    const raycaster = new THREE.Raycaster();
    const mouse = new THREE.Vector2();

    this.controls.target.set( 0, 0.5, 0 );
		this.controls.enablePan = false;
		this.controls.enableDamping = true;
    this.renderer.outputEncoding = THREE.sRGBEncoding;
    this.renderer.setSize( window.innerWidth, window.innerHeight );
    document.body.appendChild(this.renderer.domElement);
  }

  loadModel() {
    this.loaderModel.load(
      "Bedroom3.gltf",
      (obj: any) => {
        this.room = obj.scene;
        this.room.position.y = -1.5;
        this.scene.add( this.room );
      },
      (err:any) => {
        console.error('An error happened');
        console.error(err)
      }
    );
  }

  initCamera() {
    this.camera.position.z = -7;
    this.camera.position.y = 7;
    this.camera.position.x = 10;
    this.camera.lookAt(0, 0, 0);
  }

  initLight() {
    const colorBackground = new THREE.Color(0x4B0082);
    const light = new THREE.PointLight(0x6495ed, 5, 200);
    const alight = new THREE.AmbientLight(0xffffff, 1);
    const alight2 = new THREE.PointLight(0xffffff, 3, 200)
    const directionalLight = new THREE.DirectionalLight( 0xffffff, 4.5 );

    const pmremGenerator = new THREE.PMREMGenerator( this.renderer );
    this.scene.environment = pmremGenerator.fromScene( new RoomEnvironment(), 0.04 ).texture;
    light.position.set(0, 10, 0);
    alight.position.set(0, 0, 0);
    alight2.position.set(10, 10, 5);

    this.scene.background = colorBackground;
    //this.scene.background = new THREE.Color( 0xbfe3dd );
   // this.scene.add(light);
    //this.scene.add(alight);
   // this.scene.add( directionalLight );
    //this.scene.add(alight2);
  }

  animate() {
      this.controls.update();
    	requestAnimationFrame(this.animate);
      if (this.room) {
        //this.room.rotation.y += 0.001;
      }
      this.controls.update();
      this.renderer.render(this.scene, this.camera);
  }

  mounted() {
    this.initObject();
    this.loadModel();
    this.initLight();
    this.initCamera();
    this.animate();
  }
}
</script>

<style>
  * {
    margin: 0;
    padding: 0;
  }
</style>
