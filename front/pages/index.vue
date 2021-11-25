<template>
    <transition name="fade">
      <div id="popup-ampoule" v-if="popupAmpoule">
        <div id="light-icon-close" v-on:click="closePopupLight">
          <svg class="svg-icon" viewBox="0 0 20 20">
            <path d="M14.776,10c0,0.239-0.195,0.434-0.435,0.434H5.658c-0.239,0-0.434-0.195-0.434-0.434s0.195-0.434,0.434-0.434h8.684C14.581,9.566,14.776,9.762,14.776,10 M18.25,10c0,4.558-3.693,8.25-8.25,8.25c-4.557,0-8.25-3.691-8.25-8.25c0-4.557,3.693-8.25,8.25-8.25C14.557,1.75,18.25,5.443,18.25,10 M17.382,10c0-4.071-3.312-7.381-7.382-7.381C5.929,2.619,2.619,5.93,2.619,10c0,4.07,3.311,7.382,7.381,7.382C14.07,17.383,17.382,14.07,17.382,10"></path>
          </svg>
        </div>
        <div class="popup-header">
            <div>
              {{currentNameObject}}
            </div>
            <img>
        </div>
        <div class="popup-body">
          <div id="body-on-off">
            <input type="checkbox">
            <span class="slider round"></span>
          </div>
          <div id="body-color">
            <p class="title-str">Color</p>
            <input type="color" id="color-light" name="head" 
              value="#e66465">
           
          </div>
        </div>
        <div class="popup-footer">
          <button v-on:click="popupUpdateLight">Update</button>
        </div>
      </div>
    </transition>
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
import { OutlinePass } from 'three/examples/jsm/postprocessing/OutlinePass.js';
import { EffectComposer } from 'three/examples/jsm/postprocessing/EffectComposer.js';
import { FXAAShader } from 'three/examples/jsm/shaders/FXAAShader.js';
import { RenderPass } from 'three/examples/jsm/postprocessing/RenderPass.js';
import { ShaderPass } from 'three/examples/jsm/postprocessing/ShaderPass.js';
import { LuminosityShader } from 'three/examples/jsm/shaders/LuminosityShader.js';
import { UnrealBloomPass } from 'three/examples/jsm/postprocessing/UnrealBloomPass.js';
  
@Component
export default  class Index extends Vue {
  room: any = undefined;
  scene: any= undefined;
  camera: any = undefined;
  renderer: any = undefined;
  loaderModel: any= undefined;
  controls: any = undefined;
  raycaster: any = undefined;
  mouse: any = undefined;
  composer: any = undefined; 
  outlinePass: any = undefined;
  selectedObjects:any[] = [];
  currentNameObject :any = "";

  lightOffice: any= undefined;
  lightEntrie: any= undefined;
  lightKitchen: any= undefined;
  lightToilet: any=undefined;
  lightBedroom: any=undefined;
  
  popupAmpoule: any = false;

  initObject() {
    this.scene = new THREE.Scene();
		this.camera = new THREE.PerspectiveCamera(65, window.innerWidth / window.innerHeight, 0.1, 1000 );
    this.loaderModel = new GLTFLoader();
    this.renderer = new THREE.WebGLRenderer({ antialias: true });
    this.controls = new OrbitControls(this.camera, this.renderer.domElement);
    this.raycaster = new THREE.Raycaster();
    this.mouse = new THREE.Vector2();
    this.composer = new EffectComposer(this.renderer);
    const pmremGenerator = new THREE.PMREMGenerator(this.renderer);

    this.controls.target.set( 0, 2, 0 );
    this.controls.update();
    this.controls.enablePan = false;
		this.controls.enableDamping = true;
    this.renderer.outputEncoding = THREE.sRGBEncoding;
    this.renderer.setSize(window.innerWidth, window.innerHeight);
    this.scene.environment = pmremGenerator.fromScene(new RoomEnvironment(), 0.04 ).texture;
    document.body.appendChild(this.renderer.domElement);
  }

  initOutline() {
    this.composer = new EffectComposer(this.renderer);

    const renderPass = new RenderPass(this.scene, this.camera);
    this.composer.addPass(renderPass);

    this.outlinePass = new OutlinePass(new THREE.Vector2(window.innerWidth, window.innerHeight), 
      this.scene, this.camera);
    this.outlinePass.edgeStrength = Number(10);
    this.outlinePass.edgeGlow = Number(0);
    this.outlinePass.edgeThickness = Number(1);
    this.outlinePass.pulsePeriod = Number(0);
    this.outlinePass.visibleEdgeColor.set("#ffffff");
    this.outlinePass.hiddenEdgeColor.set("#000000");
    this.composer.addPass(this.outlinePass);


    const luminosityPass = new ShaderPass( LuminosityShader );
    const effectFXAA = new ShaderPass(FXAAShader);
    effectFXAA.uniforms['resolution'].value.set(1 / window.innerWidth, 1 / window.innerHeight);
    effectFXAA.renderToScreen = true;
    this.composer.addPass(effectFXAA);
  }

  closePopupLight() {
    this.popupAmpoule = false;
  }

  loadModel() {
    this.loaderModel.load(
      "Bedroom4.gltf",
      (obj: any) => {
        this.room = obj.scene;
        this.room.position.y = -5.5;
        this.room.position.x = 0;
        this.room.position.z = -15;
        this.scene.add(this.room);
        this.initCamera()
      },
      (err2:any) =>{

      },
      (err:any) => {
        console.error('An error happened');
        console.error(err)
      }
    );
  }


  initCamera() {
    this.camera.position.z = 2.56;
    this.camera.position.y = 6.9;
    this.camera.position.x = 9.9; 
    this.camera.zoom = 0.3;
    this.camera.lookAt(this.room.position);
    this.initOutline();
  }

  initLight() {
    const colorLightDefault = new THREE.Color(0x5D3FD3);
    const colorBackground = new THREE.Color(0x4B0082);
    this.lightOffice = new THREE.PointLight(0x5D3FD3, 2, 200);
    this.lightEntrie = new THREE.PointLight(0x5D3FD3, 2, 200);
    this.lightKitchen = new THREE.PointLight(0x5D3FD3, 2, 200);
    this.lightBedroom = new THREE.PointLight(0x5D3FD3, 2, 200);
    this.lightToilet = new THREE.PointLight(0x5D3FD3, 2, 200);

    this.lightOffice.position.set(6, -3, -16);
    this.lightEntrie.position.set(6, -3, 0);
    this.lightKitchen.position.set(6, -3, 16);
    this.lightBedroom.position.set(-8, -3, 0);
    this.lightToilet.position.set(-12, -3, 18);

    this.scene.background = colorBackground;
    this.scene.add(this.lightOffice);
    this.scene.add(this.lightEntrie);
    this.scene.add(this.lightKitchen);
    this.scene.add(this.lightBedroom);
    this.scene.add(this.lightToilet);
  }

  animate() {
    requestAnimationFrame(this.animate);
    if (this.room) {
      //this.room.rotation.y += 0.001;
    }
    this.controls.update();
    //this.renderer.render(this.scene, this.camera);
    this.composer.render();
  }

  listenKeyboard() {
    document.addEventListener("keydown", (event) => {
      var keyCode = event.which;
      //this.debugObject(this.lightOffice, "camera", keyCode);
    }, false);
  }

  checkIntersectedObjet(name:any) {
    const ampoules = ["AmpouleEntrie", "AmpouleToilet", "AmpouleOffice", "AmpouleKitchen",
     "AmpouleBedroom"]
    
    if (ampoules.includes(name)) {
      this.openPopupAmpoule(name);
    }
  }

  openPopupAmpoule(name: any) {
    if (this.popupAmpoule === true) return;
    let colorPicker = (<HTMLInputElement>document.getElementById("color-light"));

    console.log("Intersect", name);
    this.popupAmpoule = true;
    this.currentNameObject = name;
  }

  popupUpdateLight() {
    let popup = document.getElementById("popup-ampoule");
    let light = this.scene.getObjectByName(this.currentNameObject);

    let colorPicker = (<HTMLInputElement>document.getElementById("color-light"));
    let valuePicker = parseInt(colorPicker?.value.replace('#', '0x'), 16);

    if (this.currentNameObject === "AmpouleEntrie") {
      this.lightEntrie.color.setHex(valuePicker);
    } else if (this.currentNameObject === "AmpouleToilet") {
      this.lightToilet.color.setHex(valuePicker);
    } else if (this.currentNameObject === "AmpouleOffice") {
      this.lightOffice.color.setHex(valuePicker);
    } else if (this.currentNameObject === "AmpouleKitchen") {
      this.lightKitchen.color.setHex(valuePicker);
    } else if (this.currentNameObject === "AmpouleBedroom") {
      this.lightBedroom.color.setHex(valuePicker);
    }
    light.material.color.set(valuePicker);
    this.popupAmpoule = false;
  }

  debugObject(obj: any, str:any, keyCode: any) {
    console.log(str);
    console.log("x", obj.position.x);
    console.log("y", obj.position.y);
    console.log("z", obj.position.z);
    if (keyCode == 38) {
          obj.position.x += 2;
        } else if (keyCode == 40) {
            obj.position.x -= 2;
        } else if (keyCode == 37) {
            obj.position.z -= 2;
        } else if (keyCode == 39) {
            obj.position.z += 2;
        } else if (keyCode == 78) {
           obj.position.set(0, 0, 0);
        }
  }

  addSelectedObject(object: any) {
    this.selectedObjects = [];
    this.selectedObjects.push(object);
	}

  onMouseMove(event:any) {
	  this.mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
	  this.mouse.y = - (event.clientY / window.innerHeight) * 2 + 1;
    this.checkIntersection();
   // console.log("Mouse = ", this.mouse);
  }

  checkIntersection() {
    this.raycaster.setFromCamera(this.mouse, this.camera);
    const intersects = this.raycaster.intersectObject(this.scene, true );
    if (!this.outlinePass) return;
    if (intersects.length > 0) {
      const selectedObject = intersects[0].object;
      this.addSelectedObject(selectedObject);
      this.checkIntersectedObjet(selectedObject.name)
      this.outlinePass.selectedObjects = this.selectedObjects;
    } else {
      this.outlinePass.selectedObjects = [];
    }

  }

  mounted() {
    this.initObject();
    this.loadModel();
    this.initLight();
    this.animate();
    this.listenKeyboard();
    window.addEventListener('mousemove', this.onMouseMove, false );
  }

  init() {
  }
 }
</script>

<style  lang="scss">
  * {
    margin: 0;
    padding: 0;
  }

  #popup-ampoule {
    position: absolute;
    left:10%;
    padding: 20px;
    top:25%;
    background: white;
    border-radius: 4px;
    min-width: 200px;
    min-height: 300px;
    #light-icon-close {
        position: absolute;
        right: 10px;
        top: 10px;
        &:hover {
          fill: hotpink;
        }
        .svg-icon {
          width: 25px;
          height: 25px;
          transition: 0.5s;
          fill: inherit;
          path {
            fill: inherit;
          }
        }
    }
    .popup-header {
      margin-top: 20px;
      position:relative;
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: center;
    }
    .popup-body {
      margin-top: 40px;
      #body-on-off {
        display: flex;
        flex-direction: row;
        justify-content: left;
        align-items: center;
      }
      #body-color {
        display: flex;
        flex-direction: row;
        justify-content: left;
        align-items: center;
        .title-str {
          margin-right: 20px;
        }
      }
    }
    .popup-footer {
        margin-top: 20px;
        display: flex;
        justify-content: center;
        align-items: center;
        button {
        }
    }
  }

  .fade-enter-to, .fade-leave-active {
    opacity: 100;
  }
  .fade-enter, .fade-leave-to /* .fade-leave-active below version 2.1.8 */ {
    opacity: 0;
    transition: opacity 1.5s all;
  }


</style>
