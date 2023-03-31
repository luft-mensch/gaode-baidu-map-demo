<template>
  <div id="container"></div>
<!--  <div id="layerbox">
    <el-form :inline="true" class="demo-form-inline">
      <el-form-item label="测距">
        <el-switch
            v-model="rangFlag"
            class="ml-2"
            inline-prompt
            inactive-text="测距"
            active-text="测距"
            @change="rangFlagChange"
            style="&#45;&#45;el-switch-on-color: #13ce66; &#45;&#45;el-switch-off-color: #ff4949"
        />
      </el-form-item>
      <el-form-item label="测距">
      </el-form-item>
    </el-form>
  </div>-->
</template>

<script>

import AMapLoader from '@amap/amap-jsapi-loader';
window._AMapSecurityConfig = {
  securityJsCode: '09181faf962a60df5910fa9f4f1ddd40',
}
export default {
  name: 'GaodeMap',
  data() {
    return {
      //此处不声明 map 对象，可以直接使用 this.map赋值或者采用非响应式的普通对象来存储。
      //map:null,
      //地图加载参数
      AMapLoader:{
        "key": process.env.VUE_APP_GAODE_KEY,        // 申请好的Web端开发者Key，首次调用 load 时必填
        "version": "2.0",       // 指定要加载的 JSAPI 的版本，缺省时默认为 1.4.15
        "plugins": ['AMap.Geolocation'],            // 需要使用的的插件列表，如比例尺'AMap.Scale'等
      },
      //地图绘制参数
      //更多参数可访问 https://lbs.amap.com/api/jsapi-v2/documentation#map
      parameter:{
         zoom:11,//级别
         //创建AMap.Map对象时如果没有传入center参数，您会发现地图将自动定位到您所在城市并显示，这就是JS API的初始加载定位
         /*center: [116.397428, 39.90923],//中心点坐标*/
         viewMode:'3D', //使用3D视图
      },
      //经纬度集合
      latLngs:[],
      //测距开关，默认为关
      rangFlag:false
    }
  },
  created() {
    this.latLngs.push([113.94383,22.555423]);
    this.latLngs.push([113.949085,22.554808]);
    this.latLngs.push([113.947951,22.552864]);
  },
  //地图初始化应该在地图容器div已经添加到DOM树之后,所以放到mounted里面
  mounted(){
    this.initMap();
  },
  methods:{
    //初始化地图
    initMap(){
      AMapLoader.load(this.AMapLoader).then((AMap)=>{
        this.map = new AMap.Map('container',this.parameter);
        var geolocation = new AMap.Geolocation({
          // 是否使用高精度定位，默认：true
          enableHighAccuracy: true,
          // 设置定位超时时间，默认：无穷大
          timeout: 10000,
          // 定位按钮的停靠位置的偏移量
          offset: [20, 100],
          //  定位成功后调整地图视野范围使定位位置及精度范围视野内可见，默认：false
          zoomToAccuracy: true,
          // 定位按钮的排放位置,  RB表示右下
          position: 'RB'
        })
        this.map.addControl(geolocation);
        geolocation.getCurrentPosition((status,result)=>{
          if(status=='complete'){
            //成功的回调函数
            this.onComplete(result);
          }else{
            //失败的回调函数
            this.onError(result);
          }
        });
        // 同时引入工具条插件，比例尺插件和鹰眼插件
        AMap.plugin(['AMap.ToolBar', 'AMap.Scale', 'AMap.HawkEye', 'AMap.MapType','AMap.ControlBar'], ()=>{
          // 在图面添加工具条控件，工具条控件集成了缩放、平移、定位等功能按钮在内的组合控件
          this.map.addControl(new AMap.ToolBar());
          // 在图面添加比例尺控件，展示地图在当前层级和纬度下的比例尺
          this.map.addControl(new AMap.Scale());
          // 在图面添加鹰眼控件，在地图右下角显示地图的缩略图
          this.map.addControl(new AMap.HawkEye({isOpen:false}));
          // 在图面添加类别切换控件，实现默认图层与卫星图、实施交通图层之间切换的控制
          this.map.addControl(new AMap.MapType());
          //组合了旋转、倾斜、复位在内的地图控件。
          this.map.addControl(new AMap.ControlBar());
        });
        // 给地图实例绑定点击事件 onClick
        this.map.on('click', this.mapOnClick);
        //启动测距工具类
        this.map.plugin(["AMap.MouseTool"],()=>{
          this.ruler = new AMap.RangingTool(this.map);
        });
        this.addSegments()
      }).catch(e => {
        console.log(e);
      })
    },
    //获取定位成功的回调函数
    onComplete(result) {
        //做一些事情，比如将位置信息进行处理展示在坐标点上
      console.log(result)
    },
    //获取定位失败的回调函数
    onError(result){
      //定位失败，调用ip定位方式
      console.log(result)
      this.ipPosition()
    },
    //IP定位获取当前城市信息
    ipPosition(){
      AMap.plugin('AMap.CitySearch', ()=> {
        var citySearch = new AMap.CitySearch()
        citySearch.getLocalCity((status, result)=> {
          if (status === 'complete' && result.info === 'OK') {
            // 查询成功，result即为当前所在城市信息
            console.log(result)
            this.map.setBounds(result.bounds);
          }
        })
      })
    },
    //地图点击事件的回调函数
    mapOnClick(e){
     console.log(e.lnglat);
   },
    //测距开启或关闭
    rangFlagChange(){
      if (this.rangFlag){
        this.ruler.turnOn();
      }else {
        this.ruler.turnOff();
      }
    },
    //添加线段
    addSegments(){
      for (let i=0;i<this.latLngs.length;i++){
        if (i===0){
          this.map.add(new AMap.Marker({
            position: this.latLngs[i],
            label:{
              content: "起点"
            }
          }));
        }else {
          let str=AMap.GeometryUtil.distance(this.latLngs[i], this.latLngs[i-1]).toFixed(2)+"米"
          this.map.add(new AMap.Marker({
            position: this.latLngs[i],
            label:{
              content: str
            }
          }));
        }
      }
      this.map.add(new AMap.Polyline({
        path : this.latLngs,
      }));
    }

  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  #container {
    width: 100%;
    height: 100%;
    position: absolute;
    left: 0;
    top: 0;
  }
  #layerbox {
    position: absolute;
    right: 330px;
    top: 50px;
    background: #fff;
    z-index: 112;
    border-radius: 3px;
  }
</style>
