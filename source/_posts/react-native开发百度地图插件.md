---
title: "react-native开发百度地图插件"
date: 2017-05-03 15:18:42
tags: technology
categories: react-native
---


> 参考官网： 开发原生UI组件（[http://reactnative.cn/docs/0.43/native-component-android.html#content](http://reactnative.cn/docs/0.43/native-component-android.html#content "开发原生UI组件")）

### 一起五个步骤：

1. 创建一个ViewManager的子类。
2. 实现**createViewInstance**方法。
3. 导出视图的属性设置器：使用**@ReactProp（或@ReactPropGroup）**注解。
4. 把这个视图管理类注册到应用程序包的**createViewManagers**里。
5. 实现JavaScript模块

<!-- more -->

首先将react-native目录下的android目录用android studio打开：
{% asset_img 1.png . %}
之后就是对组件的开发过程了。

首先进入百度地图的官网，下载相应的android sdk。**先申请密钥**。
{% asset_img 2.png . %}
下载需要的jar和so文件：
{% asset_img 3.png . %}
百度地图的环境配置：官网地址（[http://lbsyun.baidu.com/index.php?title=androidsdk/guide/buildproject](http://lbsyun.baidu.com/index.php?title=androidsdk/guide/buildproject "官网地址")）

### Android Studio工程配置方法：

第一步：在工程app/libs目录下放入baidumapapi_vX_X_X.jar包，在src/main/目录下新建jniLibs目录，工程会自动加载src目录下的so动态库，放入libBaiduMapSDK_vX_X_X_X.so，注意jar和so的前3位版本号必须一致，并且保证使用一次下载的文件夹中的两个文件，不能不同功能组件的jar或so交叉使用。

第二步：工程配置还需要把jar包集成到自己的工程中，放入libs目录下。对于每个jar文件，***右键-选择Add As Library***，导入到工程中。对应在build.gradle生成工程所依赖的jar文件说明.
>jar的配置也可参考eclipse方法，进行以下操作：
菜单栏选择 File —>Project Structure。
在弹出的Project Structure 对话框中, 选择module, 然后点击 Dependencies 选项卡。

第三步： 在AndroidManifest中添加开发密钥、所需权限等信息；

（1）在application中添加开发密钥
```
<application>
  <meta-data  
    android:name="com.baidu.lbsapi.API_KEY"  
    android:value="开发者 key"/>
</application>
```

2）添加所需权限
```
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="com.android.launcher.permission.READ_SETTINGS" />
<uses-permission android:name="android.permission.WAKE_LOCK"/>
<uses-permission android:name="android.permission.CHANGE_WIFI_STATE" />
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
<uses-permission android:name="android.permission.GET_TASKS" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
<uses-permission android:name="android.permission.WRITE_SETTINGS" />
```



第四步，在应用程序创建时初始化 SDK引用的Context 全局变量：SDKInitializer.initialize(getApplicationContext());//**先省略不做**
*这一步的操作之后会以另外的方式代替*

目前的app里面的目录结构：
{% asset_img 4.png . %}

以上，android原生的百度地图开发环境已经配置完成。

现在就来创建**BaiduMapViewManager**.java和包管理类(**MyPackageManager**.java):
{% asset_img 5.png . %}

BaiduMapViewManage继承于 **SimpleViewManager<MapView>**

代码如下：
```
public class BaiduMapViewManager extends SimpleViewManager<MapView> implements BaiduMap.OnMapLoadedCallback {
    public static final String RCT_CLASS = "RCTBaiduMap";   //react-native中js来识别的标示，用于getName方法返回。
    public static final String TAG = "RCTBaiduMap";
    private ThemedReactContext mReactContext;
    public static  final int ruler = 16;
    @Override
    public String getName() {
        return RCT_CLASS;
    }
    //初始化百度地图sdk的方法，该方法在包管理类里面构造函数里面调用。重要
    public void initSDK(Context context) {
        SDKInitializer.initialize(context);
    }
    @Override
    protected MapView createViewInstance(ThemedReactContext reactContext) {
        this.mReactContext = reactContext;
        return getMap(reactContext);
    }
    //这里实例化地图的视图，需要一个context，由createViewInstance函数传入就好
    private MapView getMap(ThemedReactContext reactContext) {
        MapView mMapView = new MapView(reactContext);
        mMapView.showZoomControls(false);
        BaiduMap baiduMap = mMapView.getMap();
        baiduMap.animateMapStatus(MapStatusUpdateFactory.zoomTo(ruler), 1 * 1000);
        baiduMap.setOnMapLoadedCallback(this);
        return mMapView;   //最后返回地图
    }
    /**
     * 地图模式
     *
     * @param mapView
     * @param type
     *  1. 普通
     *  2.卫星
     */
    @ReactProp(name="mode", defaultInt = 1)
    public void setMode(MapView mapView, int type) {
        Log.i(TAG, "mode:" + type);
        mapView.getMap().setMapType(type);
    }
    /**
     * 实时交通图
     *
     * @param mapView
     * @param isEnabled
     */
    @ReactProp(name="trafficEnabled", defaultBoolean = false)
    public void setTrafficEnabled(MapView mapView, boolean isEnabled) {
        Log.d(TAG, "trafficEnabled:" + isEnabled);
        mapView.getMap().setTrafficEnabled(isEnabled);
    }
    /**
     * 实时道路热力图
     *
     * @param mapView
     * @param isEnabled
     */
    @ReactProp(name="heatMapEnabled", defaultBoolean = false)
    public void setHeatMapEnabled(MapView mapView, boolean isEnabled) {
        Log.d(TAG, "heatMapEnabled" + isEnabled);
        mapView.getMap().setBaiduHeatMapEnabled(isEnabled);
    }
    //实现接口BaiduMap.OnMapLoadedCallback的重写方法，用于地图加载后的监听。这里没有实现。
    @Override
    public void onMapLoaded() {
    }
}
```

MyReactPackage 实现于接口**ReactPackage**
代码如下：
```
public class MyReactPackage implements ReactPackage {
    private Context mContext;
    BaiduMapViewManager baiduMapViewManager;
    //书写包管理类的构造方法，参数为一个Context，该Context用于初始化百度地图的SDK，也就是ViewManager里面的initSDK方法
    public MyReactPackage(Context context) {
        this.mContext = context;
        baiduMapViewManager = new BaiduMapViewManager();
        baiduMapViewManager.initSDK(context);
    }
    @Override
    public List<NativeModule> createNativeModules(ReactApplicationContext reactContext) {
        List<NativeModule> modules=new ArrayList<>();
        //将我们创建的类添加进原生模块列表中
        modules.add(new RNModule(reactContext));
        return modules;     //所写的原生组件，在这里加入，并创建
    }
    @Override
    public List<Class<? extends JavaScriptModule>> createJSModules() {
        return Collections.emptyList();
    }
    @Override
    public List<ViewManager> createViewManagers(ReactApplicationContext reactContext) {
        return Arrays.<ViewManager>asList(
                new ReactWebViewManager(),
                new BaiduMapViewManager());   //所写的原生ViewManager类都需要在这里实例化，然后创建
    }
}
```

我们创建的packageManager都需要在**MainApplication**里面进行注册，我们只需要关注getPackages这个方法
MainApplication代码如下：

```
public class MainApplication extends Application implements ReactApplication {
  private final ReactNativeHost mReactNativeHost = new ReactNativeHost(this) {
    @Override
    public boolean getUseDeveloperSupport() {
      return BuildConfig.DEBUG;
    }
    @Override
    protected List<ReactPackage> getPackages() {
      return Arrays.<ReactPackage>asList(
          new MainReactPackage(),
          new MyReactPackage(getApplicationContext())   //在这里注册packageManager，并且传入一个Context。
      );
    }
  };
  @Override
  public ReactNativeHost getReactNativeHost() {
    return mReactNativeHost;
  }
  @Override
  public void onCreate() {
    super.onCreate();
    SoLoader.init(this, /* native exopackage */ false);
  }
}

```

**MainActivity**不需要动。

以上在android studio进行的开发完毕，现在转移到react-native里面的js开发。
创建一个模板：
比如**BaiduMap.js**
内容如下：
```
import { PropTypes } from 'react';
//在react-native里面导出requireNativeComponent
import { requireNativeComponent, View } from 'react-native';
//声明一个原生视图组件的各个参数。
var iface = {
  name: 'BaiduMap', //名字，  之后就这样使用<BaiduMap></BaiduMap>
  propTypes: {
    mode: PropTypes.number,   //根据android里面ViewManager一一对应
    trafficEnabled: PropTypes.bool,
    heatMapEnabled: PropTypes.bool,
    ...View.propTypes // 包含默认的View的属性   可以使用style等、、
  },
};
module.exports = requireNativeComponent('RCTBaiduMap', iface);   //可以理解为关联，js与java的关联。
```

使用：
**MyBaiduMap.js：**

```
import React, {Component} from 'react';
import {
  View,
  Text,
  StyleSheet
} from 'react-native';
var  BaiduMap = require('../AndroidView/BaiduMap');   //获取之前创建的模板
class BaiduMapView extends Component {
  render(){
    return (
      <View style={{flex: 1}}>
        <BaiduMap style={{height: '100%', width: '100%'}}   //注意， 最好设置高宽。
          mode={1}   //这里就是依次的props
          trafficEnabled={false}
          heatMapEnabled={false}
        ></BaiduMap>
      </View>
    );
  }
}
export default BaiduMapView;
```

百度地图初步的展示，到这里就ok了。（花了两天的时间，知道了react-native还能这样用。android studio能配合调试原生插件，只要先开启react-native服务（react-native start），然后在运行android studio的调试系统。设置断点是一样的。）。