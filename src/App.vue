<template>
    <div id="app">
        <el-container class="app-out-panel">
            <el-header class="sys-header">

                <el-button @click="loadBasemap()">{{ isBasemapText }}</el-button>
                <el-button @click="loadExample()">{{ isExampleText }}</el-button>

            </el-header>
            <el-container class="app-content-panel">
                <el-aside class="layer-control">
                    <input type="file" id="uploadFileInput" name="zip" @change="selectShpFile()" />
                </el-aside>
                <el-main class="main-map">
                    <MapView ref="MapView" />
                </el-main>
            </el-container>
        </el-container>
    </div>
</template>

<script>
import MapView from './components/Main.vue';

// eslint-disable-next-line no-unused-vars
const dialogVisible = false;

export default {
    name: 'App',
    data() {
        return {
            shpFile: '',
            geoJson: '',
            basemapCheck: false,
            enabledExample: "加载示例",
            disabledExample: "关闭实例",
            isExample: false,
            isExampleText: "加载示例",
            enabledBasemap: "显示底图",
            disabledBasemap: "隐藏底图",
            isBasemap: true,
            isBasemapText: "隐藏底图",
        }
    },
    components: {
        MapView,
    },
    methods: {
        selectShpFile() {
            this.shpFile = document.getElementById("uploadFileInput").files[0];
            console.log(this.shpFile);
            let reader = new FileReader();
            let shp = require('shpjs/dist/shp');
            let that = this;
            reader.readAsArrayBuffer(this.shpFile);
            reader.onload = function () {
                shp(this.result).then((geojson) => {
                    that.geoJson = geojson;
                    that.GeoJsonToMap();
                })
            };
        },

        GeoJsonToMap() {
            console.log(this.geoJson);
            this.$refs.MapView.geoJsonToMap(encodeURIComponent(JSON.stringify(this.geoJson)));
        },

        loadBasemap() {
            this.isBasemap = !this.isBasemap;
            this.$refs.MapView.changeBasemapVisibility();
            if (!this.isBasemap) {
                this.isBasemapText = this.enabledBasemap;
            }
            else {
                this.isBasemapText = this.disabledBasemap;
            }
        },

        loadExample() {
            this.isExample = !this.isExample;
            if (this.isExample) {
                this.isExampleText = this.disabledExample;
                this.$refs.MapView.loadExample();
            }
            else {
                this.isExampleText = this.enabledExample;
                this.$refs.MapView.unloadExample();
            }
        }
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

.layer-control {
    background-color: #c0c4cc;
}

.main-map {
    padding: 0 !important;
}
</style>

