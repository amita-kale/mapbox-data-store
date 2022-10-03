<template>
    <div>
        <div class="main-div">
            <div class="left-div">
                <table>
                    <tr v-for="(item, index) in state.backend" :key="index">
                        <td><input type="checkbox" id="namedvalue" class="m-2"
                                @change="GeoJSONDataToggle($event, index)" /></td>
                        <td> {{ item.Name}}</td>
                        <td><button class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded m-3"
                                @click="EditFun(item.Id)">Edit</button></td>
                        <td><button class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded m-3"
                                @click="removeFun(item.Id)">Delete</button></td>
                    </tr>
                </table>

            </div>
            <div class="right-div">

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
                                            Name
                                        </span></label>
                                </td>
                                <td>
                                    <input type="text"
                                        class="mt-1 px-3 py-2 bg-white border shadow-sm border-slate-300 placeholder-slate-400 focus:outline-none focus:border-sky-500 focus:ring-sky-500 block w-full rounded-md sm:text-sm focus:ring-1"
                                        v-model="state.name" placeholder="Enter name" />
                                </td>
                            </tr>
                            <tr>
                                <td>
                                    <label class="block">
                                        <span
                                            class="after:content-['*'] after:ml-0.5 after:text-red-500 block text-sm font-medium text-slate-700">
                                            Description
                                        </span></label>
                                </td>
                                <td>
                                    <input type="text"
                                        class="mt-1 px-3 py-2 bg-white border shadow-sm border-slate-300 placeholder-slate-400 focus:outline-none focus:border-sky-500 focus:ring-sky-500 block w-full rounded-md sm:text-sm focus:ring-1"
                                        v-model="state.coor" />
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
                                        class="mt-1 px-3 py-2 bg-white border shadow-sm border-slate-300 placeholder-slate-400 focus:outline-none focus:border-sky-500 focus:ring-sky-500 block w-full rounded-md sm:text-sm focus:ring-1"
                                        v-model="state.color" />
                                </td>
                            </tr>
                            <tr>
                                <td> <button type="button"
                                        class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 m-3 rounded"
                                        @click="submitForm()">
                                        Submit
                                    </button></td>
                                <td>
                                    <button type="reset"
                                        class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 m-3 rounded">
                                        Reset
                                    </button>
                                </td>
                                <td>
                                    <button type="button" @click="cancelPopup()"
                                        class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 m-3 rounded">
                                        Cancel
                                    </button>
                                </td>
                            </tr>
                        </table>
                    </form>


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
    isEdit: false,
    backend: [],
    name: '',
    color: '',
    des: {},
    coor: null,
    show: false,
    geojsonType: '',
    mapData: {} as mapboxgl.Map,
    Jsondelete: {},
});
async function removeFun(indexValue) {
    await $fetch(`http://localhost:3001/mapbox/${indexValue}`, {
        method: "DELETE"
    })
    getAPI();
}

function EditFun(indexValue) {
    state.show = true;

}

function cancelPopup() {
    state.show = false;
    state.Jsondelete.deleteAll();
}
async function submitForm() {
    state.show = false;
    let obj = {
        Name: state.name,
        Property: state.color,
        geom: state.des.geometry,
        des: state.des
    };
    await $fetch("http://localhost:3001/mapbox", {
        method: "POST",
        body: obj,
    })
        .then((res) => console.log("Data has been save successfully"))
        .catch((err) => alert(err));
    getAPI();

    state.Jsondelete.deleteAll();
}
function GeoJSONDataToggle(e, index) {
    console.log(e, index)
    console.log("Checkbox clicked", state.backend);
    if (e.target.checked == true) {

        console.log("index", state.backend[index].geom)

        state.mapData.addSource(state.backend[index].Id, {
            type: "geojson",
            data: state.backend[index].geom,
        });
        state.mapData.addLayer({
            id: state.backend[index].Id,
            source: state.backend[index].Id,
            type: "fill",
            layout: {},
            paint: { "fill-color": state.backend[index].Property, "fill-opacity": 0.5 },
        });


    } else {
        state.mapData.removeLayer(state.backend[index].Id);
        state.mapData.removeSource(state.backend[index].Id);
    }
}
async function getAPI() {
    const res: any = await $fetch(' http://localhost:3001/mapbox');
    state.backend = res;
    console.log(state.backend);

    console.log("state.backend", state.backend)
}
async function onMapLoaded(map: mapboxgl.Map) {
    getAPI();

    state.mapData = map;
    var Draw = new MapboxDraw({
        displayControlsDefault: false,
        controls: {
            polygon: true,
            trash: true,
            line_string: true,
            point: true
        },
        defaultMode: 'draw_polygon'
    });
    map.addControl(Draw, "top-right");
    state.Jsondelete = Draw;
    map.on('draw.create', updateArea);



    function updateArea(e) {
        state.show = true;
        state.geojsonType = e.features[0].geometry.type;
        state.des = e.features[0];
        state.coor = e.features[0].geometry.coordinates;
        console.log("state des", state.des);
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
</style>






