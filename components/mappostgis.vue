<template>
    <div class="main">
        <div class="left-div">
            <button class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 m-3 rounded" @click="addLayer">
                Add Layer
            </button>
        </div>
        <div class="right-div">

            <v-map :options="state.mapboxOptions" @loaded="onMapLoaded">

                <select id="droupdownid" class="dropdown">
                    <option id="streets-v11" type="radio" name="rtoggle" value="streets-v11" checked="checked">streets
                    </option>
                    <option id="light-v10" type="radio" name="rtoggle" value="light-v10">light</option>
                    <option id="dark-v10" type="radio" name="rtoggle" value="dark-v10">dark</option>
                    <option id="outdoors-v11" type="radio" name="rtoggle" value="outdoors-v11">outdoors</option>
                    <option id="satellite-v9" type="radio" name="rtoggle" value="satellite-v9">satellite</option>
                </select>
                <div class="middle-div" v-show="state.show">
                    <form>
                        <table>
                            <tr>
                                <td class="text-lg font-bold">Add geojson</td>
                            </tr>
                            <tr>
                                <td>
                                    <label class="block">
                                        <span
                                            class="after:content-['*'] after:ml-0.5 after:text-red-500 block text-sm font-medium text-slate-700">
                                            Email
                                        </span></label>
                                </td>
                                <td>
                                    <input type="text"
                                        class="mt-1 px-3 py-2 bg-white border shadow-sm border-slate-300 placeholder-slate-400 focus:outline-none focus:border-sky-500 focus:ring-sky-500 block w-full rounded-md sm:text-sm focus:ring-1"
                                        placeholder="Enter name" />
                                </td>
                            </tr>
                            <tr>
                                <td>
                                    <label class="block">
                                        <span
                                            class="after:content-['*'] after:ml-0.5 after:text-red-500 block text-sm font-medium text-slate-700">
                                            Color
                                        </span></label>
                                </td>
                                <td>
                                    <input type="color"
                                        class="mt-1 px-3 py-2 bg-white border shadow-sm border-slate-300 placeholder-slate-400 focus:outline-none focus:border-sky-500 focus:ring-sky-500 block w-full rounded-md sm:text-sm focus:ring-1" />
                                </td>
                            </tr>
                            <tr>
                                <td> <button
                                        class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 m-3 rounded">
                                        Submit
                                    </button></td>
                                <td>
                                    <button
                                        class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 m-3 rounded">
                                        Reset
                                    </button>
                                </td>
                            </tr>
                        </table>
                    </form>
                </div>


            </v-map>
        </div>

    </div>
</template>
<script setup lang="ts">
import VMap from "v-mapbox";
import mapboxgl from "mapbox-gl";
import MapboxDraw from "@mapbox/mapbox-gl-draw";
import MapboxGeocoder from "@mapbox/mapbox-gl-geocoder";

const state: any = reactive({
    mapboxOptions: {
        accessToken:
            "pk.eyJ1Ijoic29jaWFsZXhwbG9yZXIiLCJhIjoiREFQbXBISSJ9.dwFTwfSaWsHvktHrRtpydQ",
        style: "mapbox://styles/mapbox/streets-v11?optimize=true",

        center: [444.04931277036667, 26.266912177018096] as number[], //uses longitude, latitude
        zoom: 1,
        // maxZoom: 22,
    },
    map: {} as mapboxgl.Map,
    show: false,
});

function addLayer() {
    console.log("add layer method call");

    const draw = new MapboxDraw({
        displayControlsDefault: false,
        controls: {
            polygon: true,
            trash: true,
            line_string: true,
            point: true
        },
    });
    state.map.addControl(draw, "top-right");
    state.map.on('draw.create', updateArea);

    function updateArea(e) {

        state.show = true;
        // const answer = document.getElementById('calculated-area');
        // // console.log("coordinates and type show", e.features[0]);
        // //let coordinates = e.features[0].geometry.coordinates;
        // state.geojsonType = e.features[0].geometry.type;
        // state.des = e.features[0];
        // state.coor = e.features[0].geometry.coordinates;
        // answer.innerHTML = e.features[0].geometry.coordinates;

        // console.log("state des", state.des);
    }
}
function onMapLoaded(map) {
    state.map = map;

    state.map.addControl(
        new MapboxGeocoder({
            accessToken: "pk.eyJ1Ijoic29jaWFsZXhwbG9yZXIiLCJhIjoiREFQbXBISSJ9.dwFTwfSaWsHvktHrRtpydQ",
            mapboxgl: mapboxgl
        })
    );
    console.log("map loaded");
    var selectId: any = document.getElementById('droupdownid');
    selectId.addEventListener('change', event => {
        console.log(event);
        state.map.setStyle('mapbox://styles/mapbox/' + event.target.value);
    });

    state.map.addControl(new mapboxgl.FullscreenControl());
    state.map.addControl(new mapboxgl.NavigationControl());


}


</script>
<style>
.main {
    width: 100vw;
    height: 100vh;
    z-index: 1000;
}

.left-div {
    width: 20vw;
    height: 100vh;
    background-color: darkgray;
    float: left;
}

.right-div {
    width: 80vw;
    height: 100vh;
    float: right;
    z-index: 1000;

}

.dropdown {

    margin-top: 1%;
    margin-left: 2%;
    width: 10%;
    height: 5%;
    font-size: large;
    position: absolute;
    z-index: 1000;
}

.geocoder {
    background-color: aqua;
    position: absolute;
    z-index: 1;
    width: 50%;
    left: 50%;
    /* margin-left: 2%; */
    top: 10px;
}

.middle-div {

    width: 30%;
    position: absolute;
    bottom: 30%;
    right: 30%;
    background-color: aquamarine;
    padding: 15px;
    text-align: center;
    z-index: 1000;


}
</style>