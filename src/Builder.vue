<template>
  <Renderer ref="renderer" antialias orbit-ctrl resize="window">
    <Camera ref="camera" />
    <Scene ref="scene" background="#a0a0a0">
    </Scene>
  </Renderer>
</template>

<script>
// import { GridHelper,BoxGeometry,Mesh,MeshBasicMaterial,AmbientLight,TextureLoader,MeshLambertMaterial } from 'three';
import * as THREE from "three";
import { FBXLoader } from 'three/examples/jsm/loaders/FBXLoader';
import $ from 'jquery';
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
  let plane;
  let pointer, raycaster, isShiftDown = false;

  let rollOverMesh, rollOverMaterial;
  let cubeGeo, cubeMaterial;
  var cube;

  const objects = [];
  const renderer = this.$refs.renderer.renderer;
  const camera = this.$refs.camera.camera;
  camera.position.set( 500, 800, 1300 );
  camera.lookAt( 0, 0, 0 );
  const scene = this.$refs.scene.scene; 
  
  // roll-over helpers
  const rollOverGeo = new THREE.BoxGeometry( 50, 50, 50 );
  rollOverMaterial = new THREE.MeshBasicMaterial( { color: 0xff0000, opacity: 0.5, transparent: true } );
  rollOverMesh = new THREE.Mesh( rollOverGeo, rollOverMaterial );
  scene.add( rollOverMesh );


  // cubes
  cubeGeo = new THREE.BoxGeometry( 50, 50, 50 );
  cubeMaterial = new THREE.MeshLambertMaterial( { color: 0xfeb74c} );

  // grid
  const grid = new THREE.GridHelper(1000, 20);
  scene.add( grid );

  // const axesHelper = new THREE.AxesHelper( 5 );
  // scene.add( axesHelper );


	raycaster = new THREE.Raycaster();
	pointer = new THREE.Vector2();

	const geometry = new THREE.PlaneGeometry( 1000, 1000 );
	geometry.rotateX( - Math.PI / 2 );

	plane = new THREE.Mesh( geometry, new THREE.MeshBasicMaterial( { visible: false } ) );
	scene.add( plane );

	objects.push( plane );

	// lights

	const ambientLight = new THREE.AmbientLight( 0x606060 );
	scene.add( ambientLight );

    const directionalLight = new THREE.DirectionalLight( 0xffffff );
    directionalLight.position.set( 1, 0.75, 0.5 ).normalize();
    scene.add( directionalLight );

	const fbxLoader = new FBXLoader()
    fbxLoader.load(
        "./Cube.fbx",
        (object) => {
            // object.traverse(function (child) {
            //     if ((child as THREE.Mesh).isMesh) {
            //         // (child as THREE.Mesh).material = material
            //         if ((child as THREE.Mesh).material) {
            //             ((child as THREE.Mesh).material as THREE.MeshBasicMaterial).transparent = false
            //         }
            //     }
            // })
            // object.scale.set(.01, .01, .01)\
            cube = object.children[0];
            // scene.add(object)
        },
        (xhr) => {
            console.log((xhr.loaded / xhr.total) * 100 + '% loaded')
        },
        (error) => {
            console.log(error)
        }
    )

    document.addEventListener( 'pointermove', onPointerMove );
    document.addEventListener( 'pointerdown', onPointerDown );
    document.addEventListener( 'keydown', onDocumentKeyDown );
    document.addEventListener( 'keyup', onDocumentKeyUp );

    window.addEventListener( 'resize', onWindowResize );

    function saveScene() {
      var data = [];
      for(let i = 0; i < objects.length; i++) {
        data.push({"name": objects[i].name, "position": objects[i].position});
      }
      console.log( data);
      const requestURL = "http://127.0.0.1:5000/submit";
      $.ajax({
          url: requestURL,
          data: JSON.stringify(data),
          type: "POST",
          dataType: "json",
          contentType: "application/json;charset=utf-8",
          success: function(returnData){
              console.log(returnData);
          },
          error: function(xhr, ajaxOptions, thrownError){
              console.log(xhr.status);
              console.log(thrownError);
          }
      });
    }
    function onWindowResize() {

				camera.aspect = window.innerWidth / window.innerHeight;
				camera.updateProjectionMatrix();

				renderer.setSize( window.innerWidth, window.innerHeight );

				render();

			}

			function onPointerMove( event ) {

				pointer.set( ( event.clientX / window.innerWidth ) * 2 - 1, - ( event.clientY / window.innerHeight ) * 2 + 1 );

				raycaster.setFromCamera( pointer, camera );

				const intersects = raycaster.intersectObjects( objects, false );

				if ( intersects.length > 0 ) {

					const intersect = intersects[ 0 ];

					rollOverMesh.position.copy( intersect.point ).add( intersect.face.normal );
					rollOverMesh.position.divideScalar( 50 ).floor().multiplyScalar( 50 ).addScalar( 25 );

					render();

				}

			}

			function onPointerDown( event ) {

				pointer.set( ( event.clientX / window.innerWidth ) * 2 - 1, - ( event.clientY / window.innerHeight ) * 2 + 1 );

				raycaster.setFromCamera( pointer, camera );

				const intersects = raycaster.intersectObjects( objects, false );

				if ( intersects.length > 0 ) {

					const intersect = intersects[ 0 ];

					// delete cube

					if ( isShiftDown ) {

						if ( intersect.object !== plane ) {

							scene.remove( intersect.object );

							objects.splice( objects.indexOf( intersect.object ), 1 );

						}

						// create cube

					} else {
						// console.log(cube);
            // console.log(cubeGeo, cubeMaterial);
						const voxel = new THREE.Mesh( cube.geometry, cube.material );
            voxel.name = cube.name;
            voxel.scale.set( 0.5, 0.5, 0.5 );
						voxel.position.copy( intersect.point ).add( intersect.face.normal );
						voxel.position.divideScalar( 50 ).floor().multiplyScalar( 50 ).addScalar( 25 );
						scene.add( voxel );

						objects.push( voxel );
            console.log(voxel);
					}

					render();

				}

			}

			function onDocumentKeyDown( event ) {

				switch ( event.keyCode ) {

					case 16: isShiftDown = true; break;
          case 83: saveScene(); break;
				}

			}

			function onDocumentKeyUp( event ) {

				switch ( event.keyCode ) {

					case 16: isShiftDown = false; break;

				}

			}

			function render() {

				renderer.render( scene, camera );

			}
  }
};
</script>

<style>
html, body { margin:0; }
canvas { display: block; }
</style>