<template>
    <div id="map" class="map" tabindex="0">
        <div />
    </div>
</template>

<script>
import * as Proj from 'ol/proj'
import Map from 'ol/Map';
import XYZ from 'ol/source/XYZ';
import TileLayer from 'ol/layer/Tile';
import VectorLayer from 'ol/layer/Vector';
import Style from 'ol/style/Style';
import View from 'ol/View';
// import {ol} from 'ol';
import VectorSource from 'ol/source/Vector';
// import {defaults} from 'ol/control';
import GeoJSON from 'ol/format/GeoJSON';
import "ol/ol.css"
import {Stroke} from "ol/style";
import {Fill} from 'ol/style';
import ZoomToExtent from "ol/control/ZoomToExtent";

export default {
    name: 'MapView',
    data() {
        return {
            map: null,
            json: '',
            layerList: [{
                id: 0,
                name: '',
            }],
        };
    },
    mounted() {
        this.map_init();
    },
    methods: {
        map_init() {
            this.map = new Map({
                target: 'map', // 目标容器div层的id值  必备1
                layers: [new TileLayer({ // 必备2
                    source: new XYZ({
                        url: 'https://wprd0{1-4}.is.autonavi.com/appmaptile?lang=zh_cn&size=1&style=7&x={x}&y={y}&z={z}',
                    })
                })
                ],
                view: new View({ // 必备3
                    // olProj.fromLonLat方法将经纬度转换成对应的坐标
                    center: Proj.fromLonLat([113.27, 23.13]),
                    zoom: 8
                }),
            });
            this.layerList.id++;
            this.layerList.name = '高德底图';
            this.map.on('click', this.mapClick);
        },

        /*mapClick: function (){

        },*/

        geoJsonToMap(geojson) {
            this.json = JSON.parse(decodeURIComponent(geojson));
            console.log(this.json);
            let vectorSource = new VectorSource({
                features: (new GeoJSON({featureProjection: 'EPSG:3857'})).readFeatures(this.json)
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
            let zoomToExtent = new ZoomToExtent({
                extent: vectorLayer.getExtent() // 不可用？
            });
            this.map.addControl(zoomToExtent);
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

</style>
