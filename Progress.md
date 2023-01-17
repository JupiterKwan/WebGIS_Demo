# WebGIS Toolkit

## 2023.01.07 - 2023.01.13

### > 前期准备

项目基于Vue.js进行快速开发的完整的脚手架，需要先搭建Vue环境和创建新的Vue项目。

首先需要去官网下载安装nodejs，然后通过 `npm install <包名>`安装一下相应的插件。具体的前期配置在网上有相当大量的教程进行参考，比较简单。

```text
Vue/cli             （脚手架）
webpack             （用于js打包）
element-plus        （适用于Vue3，element-ui适用于Vue2）
OpenLayers          （加载地图所用的插件）
shpjs               （用于解析shapefile数据）

后续可能使用的：
storejs             （存储数据）
```

创建Vue项目可以使用如Jetbrains（付费）的IntelliJ IDEA或WebStorm来创建。不过建议使用本地安装好的nodejs和Vue/Cli（Vue3），以免出现奇怪的终端使用问题。创建项目后将自动生成Vue项目模板。

![CreateVuejs](./progress_res/屏幕截图%202023-01-13%20152035.png "CreateVuejs")

亦可以使用免费的VSCode来创建Vue项目，在终端中运行以下语句即可和前面一样生成Vue项目模板。

```cmd
    vue3使用：vue create <项目名称>
    vue2使用：vue init webpack <项目名称>
```

生成后两者皆可用`npm run dev`来启动项目。

生成后的Vue项目需要关注的文件如下：
`public`是实际的html的位置。
`node_modules`通过`npm install`安装的代码依赖库（插件的位置）
`src`**源码部分，实际上编写Vue的地方。** 其中src下的App.vue是所有Vue组件的父组件，在components下的是Vue子组件，可通过需求自行添加。
`package.json`项目配置文件。用于描述一个项目，包括我们init时的设置、开发环境、生成环境的依赖插件及版本等。

```text
├─package.json
├─src
|  ├─App.vue
|  ├─main.js
|  ├─components
|  |     └Main.vue
|  └─assets
|      └logo.png
├─public
|   ├─favicon.ico
|   └index.html
└─node_modules
```

### > Container布局容器

*插件：Element-Plus*
**注意！Element-ui适配Vue2，Element-Plus适配Vue3！不可交叉使用！**

借助Element-Plus的UI框架，在父组件App.vue中使用 `<el-container>`外层容器。同时子元素写入 `<el-header>`使容器中元素垂直布局。同时再嵌套一个 `<el-container>`，放入 `<el-aside>`、`<el-main>`，形成初步的页面布局。

```html
<template>
    <div id="app">
        <el-container class="app-out-panel">
            <el-header class="sys-header">
                <input type="file" id="uploadFileInput" name="zip" @change="selectShpFile()">
            </el-header>
            <el-container class="app-content-panel">
                <el-aside class="sys-menu">sys-menu</el-aside>
                <el-main class="main-map"><MapView ref="MapView"/></el-main>
            </el-container>
        </el-container>
    </div>
</template>
```

![container](./progress_res/屏幕截图%202023-01-13%20165257.png "container")

### > 底图显示

*插件：Openlayers 3*
在components中新建子组件Main.vue。通过观察，Vue组件的代码结构与传统三件套类似，因此主要关注在`<script>`标签中的语句。

`import`使用插件时需要先import导入要使用的库，同时也可以用import来导入其他组件的模块。
`export default`每个Vue中都有对外输出的本模块的接口。export default和export都能进行导出。但每个Vue中只允许有一个export default，export和import可以有多个。export方式导出，别的模块导入需要使用{}，而export default不需要。
`name：`在export default中创建地图，设置默认导出的组件名name为MapView。
`data()`data 必须声明为返回一个初始数据对象的函数，不过在此只是作为一个变量来使用。data中声明了两个返回值，分别为map（地图）和json（后续导入shapefile时使用）。[^1]
`mounted()`是vue中生命周期函数的一种，代表在加载html完成后执行此处[^2]
`methods:`编写函数的地方，也是实现相关功能的主要部分。

在该系统中是默认创建了高德地图的底图图层。在该组件中是通过新建一个OpenLayers插件中的Map，在Map中添加一个图层layers。高德地图的在线数据为瓦片数据，通过source.XYZ类创建瓦片图层TileLayer。在Map中添加视图View，设定地图的中心点坐标和地图级别。
由于OpenLayers的地图坐标默认为`EPSG:3857`，所以要用proj.fromLonLat类进行投影转换，将`EPSG:4326`转换为`EPSG:3857`。

```javascript
<template>
    <div id="map" class="map" tabindex="0"></div>
</template>

<script>
import * as Proj from 'ol/proj'
import Map from 'ol/Map';
import XYZ from 'ol/source/XYZ';
import TileLayer from 'ol/layer/Tile';
import View from 'ol/View';

export default {
    name: 'MapView',
    data() {
        return {
            map: null,
            json: ''
        };
    },
    mounted() {
        this.map_init();
    },
    methods: {
        map_init() {
            this.map = new Map({
                target: 'map',
                layers: [new TileLayer({
                    source: new XYZ({
                        url: 'https://wprd0{1-4}.is.autonavi.com/
                        appmaptile?lang=zh_cn&size=1&style=7
                        &x={x}&y={y}&z={z}',
                    })
                })
                ],
                view: new View({
                    // olProj.fromLonLat方法将经纬度转换成对应的坐标
                    center: Proj.fromLonLat([113.27, 23.13]),
                    zoom: 8
                }),
            })
        },
    }
}
</script>
```

地图创建好之后要在父组件App.vue引入子组件，才能使组件关联，并放到网页中。主要为三步：使用import引入子组件，以及对子组件进行注册`components`。最后在`<template>`相应位置中添加`<MapView />`即可。

```javascript
<script>
import MapView from './components/Main.vue'

export default {
    name: 'App',
    data (){
        return {
            shpFile: '',
            geoJson: '',
        }
    },
    components: {
        MapView,
    },
    methods: {

    },
};
</script>
```

![MapView](./progress_res/屏幕截图%202023-01-13%20144722.png "MapView")

### > 加载单个shapefile数据

*插件：OpenLayers、shpjs*
shapefile数据有多个文件组成（包含如`.shp, .shx, .dbf`），为了方便数据上传，采取打包`.zip`格式上传，通过shpjs插件解包处理。

在父组件的header中添加一个`<input>`标签，类型`type = 'file;'`为文件。[^3]
使用Vue的v-on绑定change事件（v-on:change可简写成@change），在用户选择文件后执行函数`selectShpFile()`。

```html
<input type="file" id="uploadFileInput" name="zip" @change="selectShpFile()">
```

在Vue3中使用`let`和`const`来声明变量。`let`用法类似于`var`，但声明只在该代码块内有效。`const`在`let`用法的基础上，声明时就需要对值初始化，且声明之后值不能改变。

在`<script>`标签中的`methods`编写`selectShpFile()`函数。[^4]

```javascript
async selectShpFile() {
    // shpFile和geoJson变量是在data()中定义的，调用时要加this，表示是在该组件中的变量
    this.shpFile = document.getElementById("uploadFileInput").files[0];
    // 使用FileReader来读取文件
    let reader = new FileReader();
    // 引入shp库，为import的另一种方式
    let shp = require('shpjs/dist/shp');
    // 以下代码中this作用域将变为function()当中。
    // 声明that等于外部的this方便对function()中的值外抛
    let that = this;
    reader.readAsArrayBuffer(this.shpFile);
    reader.onload = function () {
        shp(this.result).then((geojson) => {
            // 外抛加载的geoJson
            that.geoJson = geojson;
            that.GeoJsonToMap();
        })
    };
},

// 将加载好的geoJson传入Main.vue子组件中并绘制在地图上
GeoJsonToMap(){
    // this.$refs.（ref值）获取到的是组件实例，可以使用组件的所有方法。
    this.$refs.MapView.geoJsonToMap(encodeURIComponent(JSON.stringify(this.geoJson)));
},
```

使用`encodeURIComponent(JSON.stringify(变量))`对json进行转译，使得传参时不丢失空格等特殊字符。
传值后通过`JSON.parse(decodeURIComponent(变量))`对json进行解译使用。

在父组件加载好geoJson后调用父组件的另一函数来调用子组件的函数。这一步的目的是要等待geoJson读取完成后再进行传值。否则容易出现`this.geoJson`为空值的情况。

在子组件`Main.vue`下的`methods`编写geoJsonToMap(geojson)函数。

```javascript
import GeoJSON from 'ol/format/GeoJSON';
import VectorSource from 'ol/source/Vector';
import VectorLayer from 'ol/layer/Vector';
import Style from 'ol/style/Style';
import {Stroke} from "ol/style";

// 放入methods中
geoJsonToMap(geojson) {
    this.json = JSON.parse(decodeURIComponent(geojson));
    // 加载矢量数据来源
    let vectorSource = new VectorSource({
        features: (new GeoJSON({featureProjection: 'EPSG:3857'})).readFeatures(this.json)
    });
    // 加载矢量数据
    let vectorLayer = new VectorLayer({
        source: vectorSource,
        style: new Style({
            // 设置渲染样式
            stroke: new Stroke({
                color: 'red',
                width: 2,
            }),
        })
    })
    // 添加图层
    this.map.addLayer(vectorLayer);
},
```

![UploadSHP](./progress_res/屏幕截图%202023-01-13%20183246.png "UploadSHP")

### > 踩过的坑

1. Vue项目文件架构未理顺，父子组件位置混乱，未理解vue项目组件模式的框架。
2. html中无法直接定义元素/标签的高作为100%。如果不设定body和html的高，直接对元素/标签设置100%，会使得这几部分高度直接变成0。调用element-plus的UI框架某种程度上解决了这个问题。
3. methods中要引用data()的xxx值要使用`this.xxx`，否则会造成变量未声明或者其他问题。
4. 对于`function(e) {}`和`e => {}`实现的效果有所不同。前者内部无法再调用新的函数，后者可以。两者对于异步async的处理都未能达到预期。[^5]
5. 调用库要先导入import再使用。如果一个库里面的类中还含有多种方法，最好一个方法一个导入，使用`xxx.xxx`的引入方式容易出现问题。
6. 使用`console.log()`和网页的F12来进行Debug很有帮助。
7. 遇到不懂的，查阅各个插件的帮助文档！帮助文档！帮助文档！

### > 尚未解决的问题

以下为除上述说明中还出现的问题：

1. 对Vue组件中export default和export的理解尚不全面。
2. 加载shapefile至地图上的时间过长，经过观察耗时较长的部分为shapefile解析成geoJson、父子组件传值、生成VectorSource。
3. 目前加载shapefile仅支持单文件zip包解析、以及仅测试了上传一个shapefile文件，对于多次上传和多文件同时上传的功能尚未完善。
4. 页面样式仍需调整。

以下为上述说明中出现的问题：
[^1]: 对于`data()`的理解尚不全面。
[^2]: 在JetBrains产品下中会对`mounted()`函数警告为“未使用该函数”。但实际上地图能正常加载。在VScode中未有此警告。
[^3]: 暂时使用传统html样式，后续替换为`<el-upload>`统一UI框架。
[^4]: 对于shpjs插件的使用见官网解释。对于`FileReader()`工作机制理解尚不全面。
[^5]: 对于二者的理解尚不全面。对于异步async/await在JavaScript的使用尚不明确。
