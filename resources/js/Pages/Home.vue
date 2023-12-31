<script setup>
import GuestLayout from "@/Layouts/GuestLayout.vue";
import {onMounted, ref} from "vue";
import {Head} from '@inertiajs/vue3';
import Stats from 'three/addons/libs/stats.module.js';

import * as THREE from 'three';
import {GLTFLoader} from 'three/addons/loaders/GLTFLoader.js';
import {OrbitControls} from 'three/addons/controls/OrbitControls.js';

const objSelected = ref(null)

onMounted(() => {

    let stats;
    let camera, scene, raycaster, renderer, controls;

    let INTERSECTED;
    let theta = 0;

    const pointer = new THREE.Vector2();
    const radius = 5;

    const sceneBlock = document.getElementById('scene3D');
    const sceneRect = sceneBlock.getBoundingClientRect();

    let model3d;

    init();
    animate();


    function init() {

        camera = new THREE.PerspectiveCamera(60, window.innerWidth / window.innerHeight, 0.1, 1000);

        scene = new THREE.Scene();
        scene.background = new THREE.Color(0xf0f0f0);

        //region LIGHTs
        const light = new THREE.HemisphereLight(0xffffbb, 0xffffbb, 1);
        scene.add(light);
        //
        const dirLight = new THREE.DirectionalLight(0xffffff, 2);
        dirLight.position.set(0, 10, 0).normalize();
        scene.add(dirLight);
        //endregion

        //region load 3d model
        const loader = new GLTFLoader();
        // Optional: Provide a DRACOLoader instance to decode compressed mesh data
        // const dracoLoader = new DRACOLoader();
        // dracoLoader.setDecoderPath( '/examples/jsm/libs/draco/' );
        // loader.setDRACOLoader( dracoLoader );
        // Load a glTF resource
        loader.load(
            // resource URL
            'models/mini_market.glb',
            // called when the resource is loaded
            function (gltf) {

                scene.add(gltf.scene);

                // gltf.animations; // Array<THREE.AnimationClip>
                gltf.scene; // THREE.Group
                model3d = gltf.scenes; // Array<THREE.Group>
                // gltf.cameras; // Array<THREE.Camera>
                gltf.asset; // Object
            },
            // called while loading is progressing
            function (xhr) {

                // console.log( ( xhr.loaded / xhr.total * 100 ) + '% loaded' );

            },
            // called when loading has errors
            function (error) {

                console.log(error);

            }
        );
        //endregion

        raycaster = new THREE.Raycaster();

        renderer = new THREE.WebGLRenderer({antialias: true});
        renderer.setPixelRatio(window.devicePixelRatio);
        renderer.setSize(sceneRect.width, sceneRect.height);
        // document.body.appendChild( renderer.domElement );
        sceneBlock.appendChild(renderer.domElement);

        controls = new OrbitControls(camera, renderer.domElement);
        camera.position.set(0, 100, 20);
        controls.update();

        stats = new Stats();
        document.body.appendChild(stats.dom);

        document.addEventListener('mousemove', onPointerMove);
        document.addEventListener('mousedown', onPointerDown);

        //

        window.addEventListener('resize', onWindowResize);

    }

    function onWindowResize() {

        camera.aspect = window.innerWidth / window.innerHeight;
        camera.updateProjectionMatrix();

        renderer.setSize(window.innerWidth, window.innerHeight);

    }

    function onPointerMove(event) {
        pointer.x = ((event.clientX - sceneRect.left) / sceneRect.width) * 2 - 1;
        pointer.y = -((event.clientY - sceneRect.top) / sceneRect.height) * 2 + 1;
    }

    function animate() {

        requestAnimationFrame(animate);
        controls.update();
        render();
        stats.update();

    }

    function onPointerDown(event) {
        pointer.x = ((event.clientX - sceneRect.left) / sceneRect.width) * 2 - 1;
        pointer.y = -((event.clientY - sceneRect.top) / sceneRect.height) * 2 + 1;

        raycaster.setFromCamera(pointer, camera);

        const intersects = raycaster.intersectObjects(model3d);
        if (intersects.length > 0) {

            objSelected.value = 'Выбран:' + intersects[0].object.name
        }
    }

    function render() {

        if (typeof model3d == "undefined") return

        raycaster.setFromCamera(pointer, camera);

        const intersects = raycaster.intersectObjects(model3d);

        if (intersects?.length > 0) {

            if (INTERSECTED != intersects[0].object) {

                if (INTERSECTED) INTERSECTED.material.color.setHex(INTERSECTED.currentHex);

                INTERSECTED = intersects[0].object;
                INTERSECTED.currentHex = INTERSECTED.material.color.getHex();
                INTERSECTED.material.color.setHex(0xff0000);

            }

        } else {

            if (INTERSECTED) INTERSECTED.material.color.setHex(INTERSECTED.currentHex);

            INTERSECTED = null;

        }

        renderer.render(scene, camera);

    }

})


</script>

<template>
    <guest-layout>
        <Head title="3D scene"/>
        <div id="scene3D" class="min-h-[600px]">

        </div>
        <div class="px-4 py-2 my-6 text-center">&nbsp;{{ objSelected }}</div>
    </guest-layout>
</template>

<style scoped>

</style>
