<template>
    <div id="map" class="map" tabindex="0"></div>
    <div id="popup" class="ol-popup" v-show="this.isShow">
        <a href="#" id="popup-closer" class="ol-popup-closer" @onClick="this.popcloser"></a>
        <div id="popup-content" v-html="popupContent"></div>
    </div>
</template>

<script>
import * as Proj from 'ol/proj'
import Map from 'ol/Map';
import XYZ from 'ol/source/XYZ';
import Overlay from 'ol/Overlay';
import TileLayer from 'ol/layer/Tile';
import { toStringHDMS } from 'ol/coordinate.js';
import VectorLayer from 'ol/layer/Vector';
import Style from 'ol/style/Style';
import View from 'ol/View';
import VectorSource from 'ol/source/Vector';
import GeoJSON from 'ol/format/GeoJSON';
import "ol/ol.css"
import tileWMS from 'ol/source/TileWMS';
import { Stroke } from "ol/style";
import { Fill } from 'ol/style';
// import { map } from 'lodash';
// import {boundingExtent} from 'ol/extent';

// GCJ-02 未使用 中国地图坐标系标准
const gcj02Extent = [-20037508.342789244, -20037508.342789244, 20037508.342789244, 20037508.342789244];
const gcjMecator = new Proj.Projection({
    code: "GCJ-02",
    extent: gcj02Extent,
    units: "m"
});
Proj.addProjection(gcjMecator);

const container = document.getElementById('popup');
// const content = document.getElementById('popup-content');
const closer = document.getElementById('popup-closer');
const popOut = new Overlay({
    element: container,
    autoPan: true,
    autoPanMargin: 100,
    positioning: 'center-right',
    popupContent: null,
});

const aviMap = new TileLayer({ // 必备2
    source: new XYZ({
        url: 'https://wprd0{1-4}.is.autonavi.com/appmaptile?lang=zh_cn&size=1&style=7&x={x}&y={y}&z={z}',
        name: 'basemap',
    })
});

export default {
    name: 'MapView',
    data() {
        return {
            map: null,
            json: '',
            isShow: false,
        };
    },
    inheritAttrs: true,
    mounted() {
        this.map_init();
    },
    props: {
        layerList: [{
            id: Number,
            name: String,
        }],
    },
    methods: {
        map_init() {
            this.map = new Map({
                target: 'map', // 目标容器div层的id值  必备1
                layers: [aviMap],
                view: new View({ // 必备3
                    // olProj.fromLonLat方法将经纬度转换成对应的坐标
                    center: Proj.fromLonLat([113.27, 23.13]),
                    zoom: 8
                }),
            });
            this.map.on('click', this.mapClick);
        },

        geoJsonToMap(geojson) {
            this.json = JSON.parse(decodeURIComponent(geojson));
            console.log(this.json);
            let vectorSource = new VectorSource({
                features: (new GeoJSON({ featureProjection: 'EPSG:3857' })).readFeatures(this.json)
            });
            console.log(vectorSource);
            let vectorLayer = new VectorLayer({
                source: vectorSource,
                style: new Style({
                    stroke: new Stroke({
                        color: '#F70909',
                        width: 0.8,
                    }),
                    fill: new Fill({
                        color: 'lightCyan'
                    })
                }),
            })
            console.log(vectorLayer);
            this.map.addLayer(vectorLayer);
            this.map.getView().fit(vectorLayer.getSource().getExtent(), this.map.getSize());
        },

        mapClick(evt) {
            console.log(evt.coordinate);
            const coordinate = evt.coordinate;
            const hdms = toStringHDMS(Proj.toLonLat(coordinate));
            // content.innerHTML = '<p>You clicked here:</p><code>' + hdms + '</code>';
            // this.popupContent = '<p>You clicked here:</p><code>' + hdms + '</code>';
            this.popupContent = '<p>You clicked here:</p><code>' + hdms + '</code>';
            popOut.setPosition(coordinate);
            this.isShow = true;
            this.map.addOverlay(popOut);
        },

        popcloser() {
            popOut.setPosition(undefined);
            closer.blur();
            return false;
        },

        loadExample() {
            const baseSource = new tileWMS({
                url: 'http://localhost:8999/geoserver/wms',
                params: {
                    'LAYERS': 'ne:XZQ_D',
                    'TILED': true,
                },
                serverType: 'geoserver'
            });
            const newLayer = new TileLayer({
                source: baseSource,
                name: 'example',
            });
            this.map.addLayer(newLayer);

            newLayer.once('change', () => {
                let layerExtent = newLayer.getExtent();
                console.log("extent:", layerExtent);
                this.map.getView().fit(layerExtent);
            })


            // this.map.getView().fit("109, 20, 117, 25", this.map.getSize());
        },

        unloadExample() {
            const exampleIndex = this.map.getLayers().getArray().findIndex(layer => layer.get("name") === "example");
            this.map.getLayers().removeAt(exampleIndex);
        },

        changeBasemapVisibility() {
            this.map.getLayers().getArray().forEach((layer) => {
                if (layer.get("name") == "basemap") {
                    layer.setVisible(false);
                }})
            },
    }
    }

</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.map {
    position: relative;
    width: 100%;
    height: 100%;
}

#map:focus {
    outline: #4A74A8 solid 0.15em;
}

.ol-popup {
    position: absolute;
    background-color: white;
    box-shadow: 0 1px 4px rgba(0, 0, 0, 0.2);
    padding: 15px;
    border-radius: 10px;
    border: 1px solid #cccccc;
    bottom: 12px;
    left: -50px;
    min-width: 280px;
}

.ol-popup:after,
.ol-popup:before {
    top: 100%;
    border: solid transparent;
    content: " ";
    height: 0;
    width: 0;
    position: absolute;
    pointer-events: none;
}

.ol-popup:after {
    border-top-color: white;
    border-width: 10px;
    left: 48px;
    margin-left: -10px;
}

.ol-popup:before {
    border-top-color: #cccccc;
    border-width: 11px;
    left: 48px;
    margin-left: -11px;
}

.ol-popup-closer {
    text-decoration: none;
    position: absolute;
    top: 2px;
    right: 8px;
}

.ol-popup-closer:after {
    content: "✖";
}
</style>
