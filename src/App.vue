<template>
    <div id="app">
        <el-container class="app-out-panel">
            <el-header class="sys-header">

                <el-button text @click="dialogVisible = true">
                    click to open the Dialog
                </el-button>

                <el-dialog v-model="this.dialogVisible" title="Tips" width="30%" :before-close="handleClose()">
                    <span>This is a message</span>
                    <template #footer>
                        <span class="dialog-footer">
                            <el-button @click="this.dialogVisible = false">Cancel</el-button>
                            <el-button type="primary" @click="this.dialogVisible = false">
                                Confirm
                            </el-button>
                        </span>
                    </template>
                </el-dialog>

            </el-header>
            <el-container class="app-content-panel">
                <el-aside class="layer-control">
                    <input type="file" id="uploadFileInput" name="zip" @change="selectShpFile()" />
                    <!-- <el-checkbox-group v-model="checkLayerList">
                        <el-checkbox v-for="item in layerList" :key="item.id" :label="item.data" >
                            {{ item.data }}
                        </el-checkbox>
                    </el-checkbox-group> -->
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
import { ref } from 'vue';
import { ElMessageBox } from 'element-plus';


export default {
    name: 'App',
    data() {
        return {
            shpFile: '',
            geoJson: '',
            dialogVisible: ref(false),
        }
    },
    components: {
        MapView,
    },
    methods: {
        // const handleClose = (done: () => void) => {
        //     ElMessageBox.confirm('Are you sure to close this dialog?')
        //         .then(() => {
        //             done()
        //         })
        //         .catch(() => {
        //             // catch error
        //         })
        // },


        handleClose() {

            ElMessageBox.confirm("Are you sure you want to close?")
                .then(() => {

                })

        },

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

