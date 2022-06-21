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
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
import { FBXLoader } from 'three/examples/jsm/loaders/FBXLoader'
import Stats from 'three/examples/jsm/libs/stats.module'
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

    const light = new THREE.PointLight()
    light.position.set(0.8, 1.4, 1.0)
    scene.add(light)

    const ambientLight = new THREE.AmbientLight()
    scene.add(ambientLight)

    const camera = this.$refs.camera.camera;
    camera.position.set( 500, 800, 1300 );


    const renderer = this.$refs.renderer.renderer;

    renderer.setSize(window.innerWidth, window.innerHeight)
    document.body.appendChild(renderer.domElement)

    const controls = new OrbitControls(camera, renderer.domElement)
    controls.enableDamping = true
    controls.target.set(0, 1, 0)
    const grid = new THREE.GridHelper(1000, 20);
    scene.add( grid );
    //const material = new THREE.MeshNormalMaterial()

    const fbxLoader = new FBXLoader()
    fbxLoader.load(
        "./Untitled.fbx",
        (object) => {
            // object.traverse(function (child) {
            //     if ((child as THREE.Mesh).isMesh) {
            //         // (child as THREE.Mesh).material = material
            //         if ((child as THREE.Mesh).material) {
            //             ((child as THREE.Mesh).material as THREE.MeshBasicMaterial).transparent = false
            //         }
            //     }
            // })
            // object.scale.set(.01, .01, .01)
            scene.add(object)
        },
        (xhr) => {
            console.log((xhr.loaded / xhr.total) * 100 + '% loaded')
        },
        (error) => {
            console.log(error)
        }
    )

    window.addEventListener('resize', onWindowResize, false)
    function onWindowResize() {
        camera.aspect = window.innerWidth / window.innerHeight
        camera.updateProjectionMatrix()
        renderer.setSize(window.innerWidth, window.innerHeight)
        render()
    }

    const stats = Stats()
    document.body.appendChild(stats.dom)

    function animate() {
        requestAnimationFrame(animate)

        controls.update()

        render()

        stats.update()
    }

    function render() {
        renderer.render(scene, camera)
    }

    animate()
  }
};
</script>


<style>
html, body { margin:0; }
canvas { display: block; }
</style>