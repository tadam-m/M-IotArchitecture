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

@Component
export default  class Index extends Vue {
  objP: any= undefined;

  three() {
      const scene = new THREE.Scene();
			const camera = new THREE.PerspectiveCamera( 75, window.innerWidth / window.innerHeight, 0.1, 1000 );
      const loader = new GLTFLoader();
    
      const light = new THREE.PointLight( 0xffffff, 1, 200 );
      light.position.set( 0, 0, 0 );
      scene.add( light );

      const alight = new THREE.AmbientLight( 0xffffff, 1);
      alight.position.set( 0, 0, 0 );
      scene.add( alight );

      const alight2 = new THREE.PointLight( 0xffffff, 3, 200 )
      alight2.position.set( 0, 0, 5 );
      scene.add( alight2 );

      loader.load(
      // resource URL
      "gamerChair.gltf",

      // onLoad callback
      // Here the loaded data is assumed to be an object
      ( obj:any ) => {
        console.log("On est la")
        
        this.objP = obj.scene;
        this.objP.position.y = -1.5;
        // Add the loaded object to the scene
        scene.add( this.objP );
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
			renderer.setSize( window.innerWidth, window.innerHeight );
			document.body.appendChild( renderer.domElement );

			const geometry = new THREE.BoxGeometry();
			const material = new THREE.MeshBasicMaterial( { color: 0x00ff00 } );
			const cube = new THREE.Mesh( geometry, material );
			//scene.add( cube );

			camera.position.z = 5;

			const animate = () => {
				requestAnimationFrame( animate );

				//cube.rotation.x += 0.01;
				//cube.rotation.y += 0.01;
        if (this.objP) {
          this.objP.rotation.y += 0.01;
          //this.objP.rotation.x += 0.01;
        }
        

				renderer.render( scene, camera );
			};

			animate();
  }

  mounted() {
    this.three();
  }
}
</script>
