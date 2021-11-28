<template>
    <div>
      <PopupLight  v-if="popupAmpoule" 
      :_currentLightToggle=lightArray[iLight].lightOn 
      :_currentNameObject=lightArray[iLight].id
      :_color=lightArray[iLight].color
      :_intensity=lightArray[iLight].intensity
      v-on:popupUpdateLight=popupUpdateLight
      v-on:toggleObject=toggleObject
      v-on:closePopup=closePopup
      />
      <PopupInk v-if="popupInk" 
      v-on:closePopup=closePopup 
      :_ink=inkArray[iLight].ink
      :_substance=inkArray[iLight].substance
      :_fluid=inkArray[iLight].fluid
      :_fluidMax=inkArray[iLight].fluidMax
      v-on:popupUpdateInk=popupUpdateInk
      />

      <PopupTemp v-if="popupTemp" v-on:closePopup=closePopup  
      v-on:popupUpdateTemp=popupUpdateTemp
      :_humidity=tempArray[iLight].humidity
      :_temperature=tempArray[iLight].temperature
      />
    </div>
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
import Toggle from "~/components/Toggle.vue";
import axios from "axios";
import PopupLight from "~/components/PopupLight.vue";
import PopupInk from "../components/PopupInk.vue";

interface lightObj {
  light: THREE.PointLight;
  bulb: any;
  lightOn: boolean;
  id: string;
  deviceID: string;
  plastic: string;
  color: string;
  intensity: number;
}

interface tempObj {
  obj: any;
  temperature: number;
  deviceID: string;
  humidity: number;
  id: string;
}

interface inkObj {
  obj: any;
  ink: string;
  deviceID: string;
  substance: string;
  id: string;
  fluidMax: number;
  fluid: number;
}


@Component
export default class Three extends Vue {
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
  iLight:number = 0;
  lightArray: lightObj[] = [];
  tempArray: tempObj[] = [];
  inkArray: inkObj[] = [];
  popupAmpoule: boolean = false;
  popupInk: boolean = false;
  popupTemp: boolean = false;
  transition: string = "page";

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
    this.controls.maxPolarAngle = Math.PI / 2;
    this.controls.enablePan = false;
		this.controls.enableDamping = true;
    //this.controls.update();
    //this.renderer.outputEncoding = THREE.sRGBEncoding;
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

  toggleObject(id:any, value:any) {
    if (this.popupAmpoule) this.lightArray[this.iLight].lightOn = value;
  }

  closePopup() {
    this.popupAmpoule = false;
    this.popupInk = false;
    this.popupTemp = false;
    this.selectedObjects = [];
  }

  loadModel() {
    this.loaderModel.load(
      "Bedroom6.gltf",
      (obj: any) => {
        this.room = obj.scene;
        this.room.position.set(0, -5.5, -15);
        this.scene.add(this.room);
        this.initLight();
        this.initTemp();
        this.initInk();
        this.initCamera();
        setInterval(() => {
          this.sendTelemetryLight();
          this.sendTelemetryInk();
          this.sendTelemetryTemp();
        }, 5 * 1000)
      },
      (msg:any) =>{},
      (err:any) => {
        console.error('An error happened', err);
      }
    );
  }


  initCamera() {
    this.camera.position.set(9.9, 6.9, 2.56);
    this.camera.zoom = 0.3;
    this.camera.lookAt(this.room.position);
    this.initOutline();
  }

  initLight() {
    const colorLightDefault = new THREE.Color(0x5D3FD3);
    const colorBackground = new THREE.Color(0x4B0082);
    const lightOffice = new THREE.PointLight(0x5D3FD3, 2, 200);
    const lightEntrie = new THREE.PointLight(0x5D3FD3, 2, 200);
    const lightKitchen = new THREE.PointLight(0x5D3FD3, 2, 200);
    const lightBedroom = new THREE.PointLight(0x5D3FD3, 2, 200);
    const lightToilet = new THREE.PointLight(0x5D3FD3, 2, 200);

    lightOffice.position.set(6, -3, -16);
    lightEntrie.position.set(6, -3, 0);
    lightKitchen.position.set(6, -3, 16);
    lightBedroom.position.set(-8, -3, 0);
    lightToilet.position.set(-12, -3, 18);

    this.lightArray.push({light: lightEntrie, id: 'AmpouleEntrie', lightOn: true, bulb : this.scene.getObjectByName("AmpouleEntrie"), deviceID:"fb14bfa0-484f-11ec-b397-151de65dccd2", plastic:"LampEntrie", color:"#ffffff", intensity: 2})
    this.lightArray.push({light: lightBedroom, id: 'AmpouleBedroom', lightOn: true, bulb : this.scene.getObjectByName("AmpouleBedroom"), deviceID:"5c3f18c0-4e72-11ec-b397-151de65dccd2", plastic:"LampBedroom", color:"#ffffff", intensity: 2})
    this.lightArray.push({light: lightOffice, id: 'AmpouleOffice', lightOn: true, bulb : this.scene.getObjectByName("AmpouleOffice"), deviceID:"76ae1080-4e72-11ec-b397-151de65dccd2", plastic:"LampOffice", color:"#ffffff", intensity: 2})
    this.lightArray.push({light: lightKitchen, id: 'AmpouleKitchen', lightOn: true, bulb : this.scene.getObjectByName("AmpouleKitchen"), deviceID:"8b500c00-4e72-11ec-b397-151de65dccd2", plastic:"LampKitchen", color:"#ffffff", intensity: 2})
    this.lightArray.push({light: lightToilet, id: 'AmpouleToilet', lightOn: true, bulb : this.scene.getObjectByName("AmpouleToilet"), deviceID:"a8e85740-4e72-11ec-b397-151de65dccd2", plastic:"LampToilet",color:"#ffffff", intensity: 2})
    
    this.scene.background = new THREE.CubeTextureLoader()
    .setPath( '/' )
    .load( [
      'skybox-px.png',
      'skybox-nx.png',
      'skybox-py.png',
      'skybox-ny.png',
      'skybox-pz.png',
      'skybox-nz.png'
    ] );
      this.scene.add(lightOffice);
    this.scene.add(lightEntrie);
    this.scene.add(lightKitchen);
    this.scene.add(lightBedroom);
    this.scene.add(lightToilet);
  }

  initTemp() {
    this.tempArray.push({id:'TempEntrie', obj:this.scene.getObjectByName("TempEntrie"), temperature:0, humidity: 0, deviceID: '3ab66a70-5028-11ec-88eb-1d273089bfd9'});
    this.tempArray.push({id:'TempOutside', obj:this.scene.getObjectByName("TempOutside"), temperature:0, humidity: 0, deviceID: '49a84080-5028-11ec-88eb-1d273089bfd9'});
    this.tempArray.push({id:'TempBedroom', obj:this.scene.getObjectByName("TempBedroom"), temperature:0, humidity: 0, deviceID: '5371e5d0-5028-11ec-88eb-1d273089bfd9'});
  }

  initInk() {
    this.inkArray.push({id:'InkToilet', deviceID:'38ea4080-5029-11ec-88eb-1d273089bfd9', obj:this.scene.getObjectByName("inkToilet"),  substance:'HandWash', ink: 'Blue', fluidMax: 100, fluid: 60})
    this.inkArray.push({id:'InkOffice', deviceID:'45abc910-5029-11ec-88eb-1d273089bfd9', obj:this.scene.getObjectByName("InkOffice"),  substance:'Cream', ink: 'White', fluidMax: 100, fluid: 20})
    this.inkArray.push({id:'InkKitchen', deviceID:'4c61ecd0-5029-11ec-88eb-1d273089bfd9', obj:this.scene.getObjectByName("InkKitchen"),  substance:'Soda', ink: 'Orange', fluidMax: 100, fluid: 40})
  }

  animate() {
    requestAnimationFrame(this.animate);
    this.controls.update();
    this.composer.render();
  }

  checkIntersectedObjet(name:any) {
    if (this.popupAmpoule === true) return;

    for (let i = 0; i < this.lightArray.length; i += 1) {
      if (this.lightArray[i].id === name || this.lightArray[i].plastic === name) {
        this.iLight = i;
        this.openPopupAmpoule(name);
      }
    }
    for (let i = 0; i < this.inkArray.length; i += 1) {
      if (this.inkArray[i].id === name) {
        this.iLight = i;
        this.openPopupInk(name);
      }
    }
    for (let i = 0; i < this.tempArray.length; i += 1) {
      if (this.tempArray[i].id === name) {
        this.iLight = i;
        this.openPopupTemp(name);
      }
    }
    console.log(name);
  }

  openPopupAmpoule(name: any) {
    if (this.popupAmpoule === true) return;
    this.popupAmpoule = true;
    this.popupTemp = false;
    this.popupInk = false;
  }
  openPopupInk(name: any) {
    if (this.popupInk === true) return;
    this.popupInk = true;
    this.popupAmpoule = false;
    this.popupTemp = false;
  }
  openPopupTemp(name: any) {
    if (this.popupTemp === true) return;
    this.popupTemp = true;
    this.popupInk = false;
    this.popupAmpoule = false;
  }

  popupUpdateLight(toggled:boolean, intensity: number) {
    let colorPicker = (<HTMLInputElement>document.getElementById("color-light"));
    let valuePicker = parseInt(colorPicker?.value.replace('#', '0x'), 16);

    this.lightArray[this.iLight].intensity = intensity;
    this.lightArray[this.iLight].color = colorPicker?.value;
    this.lightArray[this.iLight].bulb.material.color.set(valuePicker);
    this.lightArray[this.iLight].light.color.setHex(valuePicker);
    this.popupAmpoule = false;
    this.powerLight(
      this.lightArray[this.iLight].light,
      this.lightArray[this.iLight].lightOn,
      this.lightArray[this.iLight].bulb,
      this.lightArray[this.iLight].intensity);
    //Send Request
    //this.sendTelemetry();
  }

  popupUpdateTemp(temperature: number, humidity: number) {
    this.tempArray[this.iLight].humidity = humidity;
    this.tempArray[this.iLight].temperature = temperature;
    this.closePopup();
  }

  popupUpdateInk(ink: string, substance : string, fluid : number, fluidMax : number) {
    this.inkArray[this.iLight].ink = ink;
    this.inkArray[this.iLight].substance = substance;
    this.inkArray[this.iLight].fluid = fluid;
    this.inkArray[this.iLight].fluidMax = fluidMax;
    this.closePopup();
  }

  async sendTelemetryLight() {
    let config = {
      headers: {
        "X-Authorization": localStorage.token,
      }
    }
    for (let i = 0; i < this.lightArray.length; i ++) {
      let data = {
        "power": this.lightArray[i].lightOn,
      }
      axios.post(
          this.$config.API_URL + 
          'api/plugins/telemetry/DEVICE/' +
          this.lightArray[i].deviceID +
          '/timeseries/ANY?scope=ANY',data, config).then((res) => {
          console.log(res);
      }).catch((e) => {
        //console.log(e);
      })
    }
  }

  async sendTelemetryInk() {
    let config = {
      headers: {
        "X-Authorization": localStorage.token,
      }
    }
    for (let i = 0; i < this.inkArray.length; i ++) {
      let data = {
        "fluid": this.inkArray[i].fluid,
        "fluidMax": this.inkArray[i].fluidMax,
      }
      axios.post(
          this.$config.API_URL + 
          'api/plugins/telemetry/DEVICE/' +
          this.inkArray[i].deviceID +
          '/timeseries/ANY',data, config).then((res) => {
          console.log(res);
      }).catch((e) => {
        //console.log(e);
      })
    }
  }

  async sendTelemetryTemp() {
    let config = {
      headers: {
        "X-Authorization": localStorage.token,
      }
    }
    for (let i = 0; i < this.tempArray.length; i ++) {
      let data = {
        "temp": this.tempArray[i].temperature,
        "humidity": this.tempArray[i].humidity,
      }
      axios.post(
          this.$config.API_URL + 
          'api/plugins/telemetry/DEVICE/' +
          this.tempArray[i].deviceID +
          '/timeseries/ANY',data, config).then((res) => {
          console.log(res);
      }).catch((e) => {
        //console.log(e);
      })
    }
  }

  powerLight(obj: any, val: boolean, light:any, intensity: number) {
    if (val == false) {
      obj.intensity = 0;
      light.material.color.set(0x000000);
    }
    else {
      obj.intensity = intensity;
    }
  }

  listenKeyboard() {
    document.addEventListener("keydown", (event) => {
      //this.debugObject(this.controls.target, "camera", event.key;);
    }, false);
  }

  debugObject(obj: any, str:any, keyCode: any) {
    console.log(str);
    console.log("x", obj.x);
    console.log("y", obj.y);
    console.log("z", obj.z);
    if (keyCode == "ArrowUp") {
      obj.x += 2;
    } else if (keyCode == "ArrowDown") {
      obj.x -= 2;
    } else if (keyCode == "ArrowLeft") {
      obj.z -= 2;
    } else if (keyCode == "ArrowRight") {
      obj.z += 2;
    } else if (keyCode == "z") {
      obj.y -= 2;
    } else if (keyCode == "s") {
      obj.y += 2;
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
  }

  checkIntersection() {
    this.raycaster.setFromCamera(this.mouse, this.camera);
    const intersects = this.raycaster.intersectObject(this.scene, true);
    if (!this.outlinePass) return;
    if (intersects.length > 0) {
      const selectedObject = intersects[0].object;
      this.addSelectedObject(selectedObject);
      if (this.checkSensor(selectedObject.name))
        this.outlinePass.selectedObjects = this.selectedObjects;
      else {
        this.outlinePass.selectedObjects = [];
      }
    } else {
      this.outlinePass.selectedObjects = [];
    }
  }

  checkSensor(name: string) {
    for (let i = 0; i < this.lightArray.length; i += 1) {
      if (this.lightArray[i].id === name || this.lightArray[i].plastic === name) {
        return true;
      }
    }
    for (let i = 0; i < this.inkArray.length; i += 1) {
      if (this.inkArray[i].id === name) {
        return true;
      }
    }
    for (let i = 0; i < this.tempArray.length; i += 1) {
      if (this.tempArray[i].id === name) {
        return true;
      }
    }
    return false;
  }

  mounted() {
    this.initObject();
    this.loadModel();
    this.animate();
    this.listenKeyboard();
    window.addEventListener('mousemove', this.onMouseMove, false);
    document.body.addEventListener('click', () => {
      if (this.selectedObjects.length > 0) this.checkIntersectedObjet(this.selectedObjects[0].name)
    })
    
  }
 }
</script>

<style  lang="scss" scoped>

.page-enter-active,
.page-leave-active {
  transition-property: opacity;
  transition-timing-function: ease-in-out;
  transition-duration: 1000ms;
}

.page-enter,
.page-leave-to {
  opacity: 0;
}
</style>
