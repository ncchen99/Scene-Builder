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
import { GLTFExporter } from 'three/examples/jsm/exporters/GLTFExporter.js';
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


    document.addEventListener( 'pointermove', onPointerMove );
    document.addEventListener( 'pointerdown', onPointerDown );
    document.addEventListener( 'keydown', onDocumentKeyDown );
    document.addEventListener( 'keyup', onDocumentKeyUp );

    window.addEventListener( 'resize', onWindowResize );


    function saveString( text, filename ) {
        const link = document.createElement( 'a' );
        link.style.display = 'none';
        document.body.appendChild( link ); // Firefox workaround, see #6594
        link.href = URL.createObjectURL( new Blob( [ text ], { type: 'text/plain' } ) );
        link.download = filename;
        link.click();
        // URL.revokeObjectURL( url ); breaks Firefox...
    }

    function Scene2GLTF(){
        // Instantiate a exporter
        const exporter = new GLTFExporter();

        // Parse the input and generate the glTF output
        exporter.parse(
            scene,
            // called when the gltf has been generated
            function ( gltf ) {

                console.log( gltf );
                const output = JSON.stringify( gltf, null, 2 );
                saveString( output, 'scene.gltf' );
            },
            // called when there is an error in the generation
            function ( error ) {
                console.log( 'An error happened' );
            }
        );
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

						const voxel = new THREE.Mesh( cubeGeo, cubeMaterial );
						voxel.position.copy( intersect.point ).add( intersect.face.normal );
						voxel.position.divideScalar( 50 ).floor().multiplyScalar( 50 ).addScalar( 25 );
						scene.add( voxel );

						objects.push( voxel );

					}

					render();

				}

			}

			function onDocumentKeyDown( event ) {

				switch ( event.keyCode ) {

					case 16: isShiftDown = true; break;
                    case 83: Scene2GLTF(); break;
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