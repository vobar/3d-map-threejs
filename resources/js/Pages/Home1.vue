<script setup>
import GuestLayout from "@/Layouts/GuestLayout.vue";
import * as THREE from 'three';
import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
import {onMounted} from "vue";
import {Head} from '@inertiajs/vue3';

onMounted(()=>{
    const sceneBlock = document.getElementById('scene3D');
    const sceneRect = sceneBlock.getBoundingClientRect();

    const scene = new THREE.Scene();
    scene.background = new THREE.Color('#ffffff')
    const camera = new THREE.PerspectiveCamera( 25, sceneRect.width / sceneRect.height, 0.1, 1000 );

    const renderer = new THREE.WebGLRenderer();

    renderer.setSize( sceneRect.width, sceneRect.height );
    sceneBlock.appendChild( renderer.domElement );

    const controls = new OrbitControls( camera, renderer.domElement );
    camera.position.set( 0, 3, 0.5 );
    controls.update();

    // const geometry = new THREE.BoxGeometry( 1, 1, 1 );
    // const material = new THREE.MeshBasicMaterial( { color: 0x00ff00 } );
    // const cube = new THREE.Mesh( geometry, material );
    // scene.add( cube );

    camera.position.z = 5;

    //region LIGHTs
    const light = new THREE.HemisphereLight( 0xffffbb, 0xffffbb, 1 );
    scene.add( light );

    const dirLight = new THREE.DirectionalLight( 0xffffff, 3000 );
    dirLight.position.set( 0, 1, 0 ).normalize();
    scene.add( dirLight );
    //endregion

    let model;

    let INTERSECTED;

    //region load 3d model
    const loader = new GLTFLoader();
    // Optional: Provide a DRACOLoader instance to decode compressed mesh data
    // const dracoLoader = new DRACOLoader();
    // dracoLoader.setDecoderPath( '/examples/jsm/libs/draco/' );
    // loader.setDRACOLoader( dracoLoader );
    // Load a glTF resource
    loader.load(
        // resource URL
        'models/divan1.glb',
        // called when the resource is loaded
        function ( gltf ) {

            model = gltf.scene

            scene.add( model );

            gltf.animations; // Array<THREE.AnimationClip>
            gltf.scene; // THREE.Group
            gltf.scenes; // Array<THREE.Group>
            gltf.cameras; // Array<THREE.Camera>
            gltf.asset; // Object

        },
        // called while loading is progressing
        function ( xhr ) {

            // console.log( ( xhr.loaded / xhr.total * 100 ) + '% loaded' );

        },
        // called when loading has errors
        function ( error ) {

            console.log( 'An error happened' );

        }
    );
    //endregion
    //region RAYCASTING
    const raycaster = new THREE.Raycaster();
    const pointer = new THREE.Vector2();

    function onPointerMove( event ) {

        // calculate pointer position in normalized device coordinates
        // (-1 to +1) for both components

        pointer.x = ( event.clientX / sceneRect.width ) * 2 - 1;
        pointer.y = - ( event.clientY / sceneRect.width ) * 2 + 1;

    }

    function animate() {
        requestAnimationFrame( animate );

        controls.update();

        renderer.render( scene, camera );
        render();
    }
    animate();
    function render() {

        if (typeof model == "undefined") return

        // const intersects = raycaster.intersectObjects( scene.children );
        //
        // for ( let i = 0; i < intersects.length; i ++ ) {
        //
        //     intersects[ i ].object.material.color.set( 0xff0000 );
        //
        // }
        //
        // renderer.render( scene, camera );

        raycaster.setFromCamera( pointer, camera );

        // console.log(model)

        const intersects = raycaster.intersectObjects( model );

        if ( intersects.length > 0 ) {
            if ( INTERSECTED !== intersects[ 0 ].instanceId ) {

                if ( INTERSECTED ) INTERSECTED.material.color.setHex( INTERSECTED.currentHex );

                INTERSECTED = intersects[ 0 ].object;
                INTERSECTED.currentHex = INTERSECTED.material.color.getHex();
                INTERSECTED.material.color.setHex( 0xff0000 );

            }

        } else {

            if ( INTERSECTED ) INTERSECTED.material.color.setHex( INTERSECTED.currentHex );

            INTERSECTED = null;

        }

        renderer.render( scene, camera );


    }

    window.addEventListener( 'pointermove', onPointerMove );

    window.requestAnimationFrame(render);
    //endregion
})


</script>

<template>
    <guest-layout>
        <Head title="3D scene" />
        <div id="scene3D" class="min-h-[400px]">

        </div>
    </guest-layout>
</template>

<style scoped>

</style>
