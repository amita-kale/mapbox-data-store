<template>
    <div>

        <div class="main-div">
            <div class="left-div">

                <ul id="checkboxId">
                    <li v-for="(item, index) in state.layers" :key="index">
                        <input type="checkbox" id="namedvalue" @change="GeoJSONDataToggle($event, index)" />
                        {{ item.name}}
                    </li>

                </ul>
            </div>
            <div class="right-div">
                <div class="calculation-box">
                    <p>Click the map to draw a polygon.</p>
                    <div id="calculated-area"></div>
                </div>
                <div class="middle-div" v-show="state.show">
                    Name <input type="text" id="name" v-model="states.obj.name"><br>
                    <!-- description<input type="text" id="des" v-model="states.obj.des"> -->
                    <button class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded"
                        @click="submitForm()">Submit</button>
                </div>

                <v-map :options="state.map" @loaded="onMapLoaded">

                </v-map>
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
    },

    layers: [],
    show: false,

});
const states: any = reactive({
    obj: {
        name: '',
        des: ''
    }

});
function submitForm() {

    state.layers.push(states.obj);
    console.log("submit date" + state.layers);
    state.show = false;
}


function GeoJSONDataToggle(e, index) {
    console.log(e, index)
    console.log("submit toggle" + state.layers);
    if (e.target.checked == true) {
        console.log("id", state.layers[index].state.id)
        state.map.addSource(state.layers[index].state.id, {
            type: "geojson",
            data: state.layers[index].state,
        });
        state.map.addLayer({
            id: state.layers[index].state.id,
            source: state.layers[index].state.id,
            type: "fill",
            layout: {},
            paint: { "fill-color": "blue", "fill-opacity": 0.5 },
        });
    } else {
        state.map.removeLayer(state.layers[index].state.id);
        state.map.removeSource(state.layers[index].state.id);
    }

}


function onMapLoaded(map: mapboxgl.Map) {

    var Draw = new MapboxDraw();
    map.addControl(Draw, "top-right");

    map.on('draw.create', updateArea);
    // map.on('draw.delete', geojsondelete);
    //map.on('draw.update', updateArea);


    function updateArea(e) {
        state.show = true;
        // const answer = document.getElementById('calculated-area');
        // console.log("coordinates and type show", e.features[0]);
        // let coordinates = e.features[0].geometry.coordinates;
        // let geojsonType = e.features[0].geometry.type;
        //states.obj.des = e.features[0];
        // console.log("gh", coordinates);
        // console.log("gh", geojsonType);

        // answer.innerHTML = e.features[0].geometry.type + e.features[0].geometry.coordinates;

        let data1: any = {
            state: e.features[0],
        };
        state.layers.push(data1);
        console.log("state.layes", state.layers)
    }
}
</script>
<style>
.left-div {
    height: 100vh;
    width: 20vw;
    background-color: azure;
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

label {

    font-size: large;
}

ul {
    list-style-type: none;
}


.calculation-box {
    height: 10%;
    width: 20%;
    position: absolute;
    bottom: 40px;
    right: 10px;
    background-color: rgba(255, 255, 255, 0.9);
    padding: 15px;
    text-align: center;
    z-index: 1000;
    /* display: inline-block; */

}

.middle-div {
    /* height: 10%; */
    width: 20%;
    position: absolute;
    bottom: 30%;
    right: 30%;
    background-color: aquamarine;
    padding: 15px;
    text-align: center;
    z-index: 1000;
    /* display: inline-block; */

}

input {
    margin: 5%;
    padding: 3%;
}


p {
    font-family: 'Open Sans';
    margin: 0;
    font-size: 13px;
    /* display: inline-block; */
}

.mapboxgl-popup {
    max-width: 400px;
    font: 12px/20px 'Helvetica Neue', Arial, Helvetica, sans-serif;
}
</style>