<template>
    <div>

        <div class="main-div">
            <div class="left-div">Button</div>
            <div class="right-div">
                <v-map :options="state.map" @loaded="onMapLoaded"></v-map>
            </div>
        </div>


    </div>
</template>
<script setup lang="ts">
import VMap from "v-mapbox";
import mapboxgl from "mapbox-gl";
import MapboxDraw from "@mapbox/mapbox-gl-draw";
const state: any = reactive({
    map: {
        accessToken:
            "pk.eyJ1Ijoic29jaWFsZXhwbG9yZXIiLCJhIjoiREFQbXBISSJ9.dwFTwfSaWsHvktHrRtpydQ",
        style: "mapbox://styles/mapbox/streets-v11?optimize=true",

        center: [444.04931277036667, 26.266912177018096] as number[], //uses longitude, latitude
        zoom: 1,
        // maxZoom: 22,


    }
});

function onMapLoaded(map: mapboxgl.Map) {

    var Draw = new MapboxDraw();
    map.addControl(Draw, "top-right");

}
</script>
<style>
.left-div {
    height: 100vh;
    width: 20vw;
    background-color: aquamarine;
    float: left;
}

.right-div {
    height: 100vh;
    width: 80vw;
    float: right;

}

.main-div {
    height: 100vh;
    width: 100vw;
}
</style>