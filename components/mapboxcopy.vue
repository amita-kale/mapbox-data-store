<template>
    <div>

        <div class="main-div">
            <div class="left-div">

                <ul id="checkboxId">
                    <li v-for="(item, index) in state.layers" :key="index">
                        <input type="checkbox" id="namedvalue" @change="GeoJSONDataToggle($event, index)" />
                        {{ item.name}}
                        <button class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded"
                            @click="removeFun(index)">Delete</button>
                    </li>
                    <button class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded"
                        id="zoom">Zoom</button>
                    <!-- <button class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded" @click="flytoFunction()">Flyto</button> -->

                    <div class="map-overlay top">
                        <div class="map-overlay-inner">
                            <label>Layer opacity: <span id="slider-value">100%</span></label>
                            <input id="slider" type="range" min="0" max="100" step="0" value="100">
                        </div>
                    </div>
                </ul>
            </div>
            <div class="right-div">
                <div class="calculation-box">
                    <p>Click the map to draw a polygon.</p>
                    <div id="calculated-area"></div>
                </div>
                <div class="middle-div" v-show="state.show">
                    Name <input type="text" id="name" v-model="state.name"><br>
                    description<input type="text" id="des" v-model="state.des">
                    color:<input type="color" id="favcolor" name="favcolor" v-model="state.color">
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
    name: '',
    color: '',
    des: {},
    coor: null,
    show: false,
    geojsonType: '',
    obj: {
        name: '',
        des: {},
        color: ''
    },
    mapData: {} as mapboxgl.Map,
    Jsondelete: {},

});

function removeFun(i) {
    state.layers.splice(i, 1);
    state.mapData.removeLayer(state.layers[i].des.id);
    state.mapData.removeSource(state.layers[i].des.id);
}

function submitForm() {
    const obj = {
        name: state.name,
        des: state.des,
        color: state.color,
        // coor: state.coor
    }
    console.log("coor:", state.coor, "des", state.des)
    state.layers.push(obj);
    this.obj = {
        des: '',
        color: '',
        name: '',
    }
    console.log("submit data", state.layers);
    state.show = false;
    state.Jsondelete.deleteAll();


}

// flytoFunction(){
//   //flyto method take to the provided LngLat

// }

function GeoJSONDataToggle(e, index) {
    console.log(e, index)
    console.log("submit toggle", state.layers);
    if (e.target.checked == true) {
        // console.log("coordinate polygon", state.layers[index].des.geometry.coordinates);

        let colorPick;


        let colorSelect = state.layers.filter((e) => {
            if (e.des.id == state.layers[index].des.id) {
                colorPick = e.color;
                return e.color;
            }
        });

        console.log("id", state.layers[index].des.id)
        state.mapData.addSource(state.layers[index].des.id, {
            type: "geojson",
            data: state.layers[index].des,
        });
        state.mapData.addLayer({
            id: state.layers[index].des.id,
            source: state.layers[index].des.id,
            type: "fill",
            layout: {},
            paint: { "fill-color": colorPick, "fill-opacity": 0.5 },
        });
        // let flylocation = state.layers[index].coor[0]
        let flylocation = state.coor[0];
        console.log("line coor", flylocation);

        if (state.geojsonType == 'LineString') {
            state.mapData.flyTo({
                center: flylocation,
                zoom: 9,
                speed: 0.9,
                curve: 1,
                easing(t) {
                    return t;
                }
            });
        }
        else {
            state.mapData.flyTo({
                center: flylocation[0],
                zoom: 9,
                speed: 0.9,
                curve: 1,
                easing(t) {
                    return t;
                }
            });
        }
    } else {
        state.mapData.removeLayer(state.layers[index].des.id);
        state.mapData.removeSource(state.layers[index].des.id);
    }

}


function onMapLoaded(map: mapboxgl.Map) {
    state.mapData = map;
    var Draw = new MapboxDraw();
    map.addControl(Draw, "top-right");
    state.Jsondelete = Draw;
    map.on('draw.create', updateArea);
    // map.on('draw.delete', geojsondelete);
    //map.on('draw.update', updateArea);




    document.getElementById('zoom').addEventListener('click', () => {
        map.zoomTo(18, { duration: 9000 });
    });

    map.setPaintProperty('state.layers[index].des.id', 'fill-color', [
        'interpolate',
        // Set the exponential rate of change to 0.5
        ['exponential', 0.5],
        ['zoom'],
        // When zoom is 1, buildings will be beige.
        15,
        '#D9D3C9',
        // When zoom is 18 or higher, buildings will be yellow.
        18,
        '#ffd700'
    ]);
    const slider = document.getElementById('slider');
    const sliderValue = document.getElementById('slider-value');
    slider.addEventListener('input', (e: any) => {
        // Adjust the layers opacity. layer here is arbitrary - this could
        // be another layer name found in your style or a custom layer
        // added on the fly using `addSource`.
        map.setPaintProperty(
            'state.layers[index].des.id',
            'raster-opacity',
            parseInt(e.target.value, 10) / 100
        );

        // Value indicator
        sliderValue.textContent = e.target.value + '%';
    });


    function updateArea(e) {

        state.show = true;
        const answer = document.getElementById('calculated-area');
        // console.log("coordinates and type show", e.features[0]);
        //let coordinates = e.features[0].geometry.coordinates;
        state.geojsonType = e.features[0].geometry.type;
        state.des = e.features[0];
        state.coor = e.features[0].geometry.coordinates;
        answer.innerHTML = e.features[0].geometry.coordinates;
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

#zoom {
    display: block;
    position: relative;
    margin: 20px auto;
    width: 50%;
    height: 40px;
    padding: 10px;
    border: none;
    border-radius: 3px;
    font-size: 12px;
    text-align: center;
    color: #fff;
    background: #ee8a65;
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