<template>
    <MapView ref="mapview" />
    <div id="app">
        <el-container class="app-out-panel">
            <el-header class="sys-header">
<!--                <el-upload ref="upload" :limit="10" accept="zip" :auto-upload="true" :file-list="fileList" :on-change="loadFile" :before-upload="beforeUpload">
                    <el-button type="primary" size="small">叠加shp</el-button>
                </el-upload>-->

                <input type="file" id="uploadFileInput" name="zip" @change="selectShpFile()">
                
            </el-header>
            <el-container class="app-content-panel">
                <el-aside class="sys-menu">
                    <el-checkbox-group v-model="this.$refs.mapview.layerList">
                        <el-checkbox v-for="item in this.$refs.mapview.layerList" :key="item.id" :label="item.name">
                            {{ item.name }}
                        </el-checkbox>
                    </el-checkbox-group>
                </el-aside>
                <el-main class="main-map"><MapView ref="MapView"/></el-main>
            </el-container>
        </el-container>
    </div>
</template>

<script>
import MapView from './components/Main.vue'
// import store from 'storejs'
// import * as shapefile from "shapefile"

export default {
    name: 'App',
    data (){
        return {
            // fileList: [],
            shpFile: '',
            geoJson: '',
        }
    },
    provide(){
        return{
            geoJson: encodeURIComponent(JSON.stringify(this.geoJson)),
        }
    },
    components: {
        MapView,
    },
    methods: {
        async selectShpFile() {
            this.shpFile = document.getElementById("uploadFileInput").files[0];
            console.log(this.shpFile);
            let reader = new FileReader();
            let shp = require('shpjs/dist/shp');
            let that = this;
            reader.readAsArrayBuffer(this.shpFile);
            reader.onload = function () {
                shp(this.result).then((geojson) => {
                    that.geoJson = geojson;
                    console.log(this.geoJson);
                    that.GeoJsonToMap();
                })
            };
        },

        GeoJsonToMap(){
            this.$refs.MapView.geoJsonToMap(encodeURIComponent(JSON.stringify(this.geoJson)));
        }

        // beforeUpload(file){
        //     const isFile = (file.raw.type === 'zip');
        //     if (!isFile){
        //         this.$message.error('仅上传zip格式文件');
        //     }
        //     console.log(file.raw.data());
        //     return isFile;
        // },

        // loadFile(){
        //     const shapefile = require('shpjs/dist/shp');
        //     let fileReader = new FileReader();
        //     fileReader.readAsArrayBuffer(this.fileList);
        //     fileReader.onload = function () {
        //         shapefile.parseZip(this.result).then((geojson) => {
        //             console.log("loadzip", geojson);
        //         })
        //     }
        // },
    },
};
</script>

<style>
html,
body,
#app {
    position: relative;
    width: 100%;
    height: 100%;
    margin: 0;
}

.app-out-panel,
.app-content-panel {
    height: 100%;
}

.sys-header {
    background-color: #303133;
    line-height: 60px;
    height: 60px;
    color: #fff;
    font-size: 24px;
}

.sys-menu {
    background-color: #c0c4cc;
}

.main-map {
    padding: 0 !important;
}
</style>

