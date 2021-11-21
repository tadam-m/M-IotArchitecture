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

@Component
export default  class Index extends Vue {
  objP: any= undefined;
  scene: any= undefined;
  three() {
      this.scene = new THREE.Scene();
			const camera = new THREE.PerspectiveCamera( 75, window.innerWidth / window.innerHeight, 0.1, 1000 );
      const loader = new GLTFLoader();
      const colorBackground = new THREE.Color(0x5D3FD3);
      this.scene.background = colorBackground;
      const light = new THREE.PointLight( 0xffffff, 1, 200 );
      light.position.set( 0, 10, 0 );
      this.scene.add( light );

      const alight = new THREE.AmbientLight( 0xffffff, 1);
      alight.position.set( 0, 0, 0 );
      this.scene.add( alight );

      const alight2 = new THREE.PointLight( 0xffffff, 3, 200 )
      alight2.position.set( 10, 10, 5 );
      this.scene.add( alight2 );

      loader.load(
      // resource URL
      "bedroom.gltf",

      // onLoad callback
      // Here the loaded data is assumed to be an object
      ( obj:any ) => {
        console.log("On est la")
        
        this.objP = obj.scene;
        this.objP.position.y = -1.5;
        // Add the loaded object to the scene
        this.scene.add( this.objP );
      },

      // onProgress callback
      function ( xhr:any ) {
        console.log( (xhr.loaded / xhr.total * 100) + '% loaded' );
      },

      // onError callback
      function ( err:any ) {
        console.error( 'An error happened' );
        console.error(err)
      }
    );
      
			const renderer = new THREE.WebGLRenderer();

      const controls = new OrbitControls(camera, renderer.domElement);
      controls.update();
			renderer.setSize( window.innerWidth, window.innerHeight );
			document.body.appendChild( renderer.domElement );

			const geometry = new THREE.BoxGeometry();
			const material = new THREE.MeshBasicMaterial( { color: 0x00ff00 } );
			const cube = new THREE.Mesh( geometry, material );
			//scene.add( cube );

			camera.position.z = -7;
      camera.position.y = 7;
      camera.position.x = 10;
      camera.lookAt(0, 0, 0);

			const animate = () => {
				requestAnimationFrame( animate );

				//cube.rotation.x += 0.01;
				//cube.rotation.y += 0.01;
        if (this.objP) {
          this.objP.rotation.y += 0.001;
          //this.objP.rotation.x += 0.01;
        }
        

				renderer.render(this.scene, camera );
			};

			animate();
  }

  mounted() {
    this.three();
  }
}
</script>
