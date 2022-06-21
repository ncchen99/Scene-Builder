<template>
  <Renderer ref="renderer" antialias orbit-ctrl resize="window">
    <Camera ref="camera" />
    <Scene ref="scene" background="#a0a0a0">
    </Scene>
  </Renderer>
</template>

<script>
// import { GridHelper,BoxGeometry,Mesh,MeshBasicMaterial,AmbientLight,TextureLoader,MeshLambertMaterial } from 'three';
import * as THREE from 'three'
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js';
import {
  Camera,
  DirectionalLight,
  FbxModel,
  HemisphereLight,
  Renderer,
  PhongMaterial,
  Plane,
  Scene,
} from 'troisjs';
export default {
  components: {
    Camera,
    DirectionalLight,
    FbxModel,
    HemisphereLight,
    Renderer,
    PhongMaterial,
    Plane,
    Scene,
  },
  mounted() {
    const scene = this.$refs.scene.scene; 
    scene.add(new THREE.AxesHelper(5))

    const ambientLight = new THREE.AmbientLight()
    scene.add(ambientLight)

    let pointLight = new THREE.PointLight(0xeeff00)
    pointLight.position.set(800, 800, 800)
    pointLight.castShadow = true
    scene.add(pointLight)
    let pointLightHelper = new THREE.PointLightHelper(pointLight)
    scene.add(pointLightHelper)

    const camera = this.$refs.camera.camera;
    camera.position.set( 500, 800, 1300 );


    const renderer = this.$refs.renderer.renderer;

    renderer.setSize(window.innerWidth, window.innerHeight)
    document.body.appendChild(renderer.domElement)

    const grid = new THREE.GridHelper(1000, 20);
    scene.add( grid );
    //const material = new THREE.MeshNormalMaterial()
    const loader = new GLTFLoader();

    loader.load( './Grass.gltf', function ( gltf ) {
        gltf.scene.scale.set(1000, 1000, 1000);
        scene.add( gltf.scene );
    }, function ( xhr ) {
		console.log( ( xhr.loaded / xhr.total * 100 ) + '% loaded' );
	},
	// called when loading has errors
	function ( error ) {
		console.log( 'An error happened' );
	} );
  }
};
</script>


<style>
html, body { margin:0; }
canvas { display: block; }
</style>
