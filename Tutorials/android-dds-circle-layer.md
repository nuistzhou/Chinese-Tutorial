---
title: 针对安卓的数据驱动样式
description: 创建一张针对安卓的地图，并基于属性数据设计一个圆的样式。

thumbnail: androidDdsCircleLayer
level: 2
topics:
- mobile apps
language:
- Java
前提: 一个设置好地图视图的安卓应用程序，并且熟悉 Android Studio 和 Java。
prependJs:
  - "import * as constants from '../../constants';"
  - "import Icon from '@mapbox/mr-ui/icon';"
  - "import Note from '@mapbox/dr-ui/note';"
  - "import BookImage from '@mapbox/dr-ui/book-image';"
  - "import { AndroidTutorialCodeBlock } from '../../components/android-tutorial-code-block';"
  - "import * as snippet from '../../snippets/android-dds-circle-layer.js'"
  - "import UserAccessToken from '../../components/user-access-token';"
  - "import Button from '@mapbox/mr-ui/button';"
contentType: tutorial
---

[数据驱动样式](/help/glossary/data-driven-styling/) 是 Mapbox Maps SDK for Android 的一个强大特性，其允许您使用属性数据来设计地图样式。通过数据驱动样式, 您可以全自动地设计基于各自属性的地图要素样式。 在本教程中, 您将创建一张包含圆形图层基于Android的地图，其依据属性数据设计样式。

<div class='align-center'>
<img src='/help/img/android/android-dds-style-by-attribute.png' alt='map with data styled by attribute on an Android device' class='inline wmax360-mm wmax-full'>
</div>

## 开始

本教程假定您已经熟悉Java和Android Studio。以下是您开始前需要的一些资料：

- **一个包含Mapbox Maps SDK for Android的应用程序。** 本教程假定您已经开始创建一个基于Mapbox Maps SDK for Android的安卓应用程序。如果Maps SDK for Android对您来说还是陌生的，请先完成教程[Mapbox Maps SDK for Android起步](/help/tutorials/first-steps-android-sdk/)来创建一个地图视图。
- **数据。** 我们从哥伦比亚特区的[Open Data DC](http://opendata.dc.gov/)收集了其行道树的位置数据。每一棵树都有一个`DBH`属性，代表[树胸高直径](https://en.wikipedia.org/wiki/Diameter_at_breast_height), 用来衡量树的大小.

{{
<Button href="/help/data/street-trees-DC.zip" passthroughProps={{ download: "street-trees-DC" }} >
    <Icon name='arrow-down' inline={true} /> Download Shapefile
</Button>
}}

## 上传数据到Mapbox

在本教程中，您将使用[向量瓦片集](/help/glossary/tileset) 来在您的应用中展示数据。您可以通过上传Open Data DC的Shapefile到Mapbox Studio来创建向量瓦片集：

1. 登录[Mapbox Studio](https://www.mapbox.com/studio)。
1. 访问[瓦片集页面](https://www.mapbox.com/studio/tilesets)。
1. 点击 **新瓦片集**。
1. 选择您之前下载好的Shapefile并点击**确认**。
1. 右下角将出现一个弹出框，显示上传进度。
1. 一旦上传 _完成_, 瓦片集即可使用。点击弹出框里的瓦片集名称，会打开该瓦片集的详情页面。
1. 请记录该详情页面右侧的**瓦片集 ID**。稍后，您将使用该ID来添加该瓦片集到您的应用程序。

## 初始化一个地图视图

在Android Studio中创建一个新项目并初始化一个地图视图。为使用Android Studio项目创建一个Mapbox地图，并添加定制化数据来使用数据驱动样式来设计其样式，您需要以下5个文件：

- **build.gradle**: Android Studio使用Gradle工具集将源文件和源代码编译成一个APK文件。`build.gradle` 文件被用来配置构建和管理包括Mapbox Maps SDK for Android在内的依赖。
- **AndroidManifest.xml**: 您可以在`AndroidManifest.xml` 文件中描述应用程序的组件，比如同Mapbox相关的权限。
- **activity_main.xml**: 您可以在`activity_main.xml` 文件中设置地图视图的属性(例如地图视图的中心, 缩放级别以及地图样式)。
- **strings.xml**: 您可以将access token存储在`strings.xml`文件中。
- **MainActivity.java**: 您可以在`MainActivity.java`文件中指定Mapbox的各种交互.

{{
  <div className="txt-s txt-fancy mb6" style={{ color: "#273d56" }}>build.gradle (App module)</div>
}}

```groovy
// in addition to the rest of your build.gradle contents
// you should include the following repository and dependency
repositories {
    mavenCentral()
}

dependencies {
    implementation 'com.mapbox.mapboxsdk:mapbox-android-sdk:{{constants.VERSION_ANDROID_MAPS}}'
}
```

{{
  <div className="txt-s txt-fancy mb6" style={{ color: "#273d56" }}>Manifest.xml</div>
}}

```xml
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
```



{{
  <AndroidTutorialCodeBlock
    filename="activity_main.xml"
    code={snippet.finalLayout}
  />
}}

{{
  <div className="txt-s txt-fancy mb6" style={{ color: "#273d56" }}>strings.xml</div>
}}

```xml
<string name="access_token" translatable="false">{{ <UserAccessToken /> }}</string>
```

{{
  <AndroidTutorialCodeBlock
    filename="MainActivity.java"
    code={snippet.finalJava}
    copyRanges={[
      [1,10],
      [23,40],
      [63,67],
      [70,111]
    ]}
  />
}}

You can learn how to set up an Android Studio project with the Maps SDK for Android in the [First steps with the Mapbox Maps SDK for Android](/help/tutorials/first-steps-android-sdk/) guide.

Run your application, and you should see a map in the Mapbox Dark style centered on Logan Circle.

<div class='align-center'>
<img src='/help/img/android/android-dds-initialize-map.png' alt='initialized map on an Android device' class='wmax360'>
</div>

## Load the source

Next, you'll load the tileset that you added to your Mapbox Studio account into the application using the [tileset ID](/help/glossary/tileset-id). Find your tileset ID on the [Tilesets page in Mapbox Studio](https://www.mapbox.com/studio/tilesets/) by clicking the {{<Icon name='menu' inline={true} />}} next to the tileset you uploaded.

First you'll need to import the correct classes in the `MainActivity.java` file. This will allow you to add a vector source and create a circle layer from that data.

Then, add the code that pulls in the data in your tileset and adds it as a layer to your map. In the `MainActivity.java` file, inside `public void onStyleLoaded(@NonNull Style style) { ... }` add the following code:

{{
  <AndroidTutorialCodeBlock
    filename="MainActivity.java"
    code={snippet.finalJava}
    copyRanges={[
      [12,13],
      [41,50],
      [62,62]
    ]}
  />
}}


Rerun your application and you'll see the data from your tileset displayed on the dark map. Notice that the black circles are difficult to see on top of the dark map style. Next, you'll style the data to be more visible.

<div class='align-center'>
<img src='/help/img/android/android-dds-load-data.png' alt='map with data on an Android device' class='wmax360'>
</div>

## Style the layer

Now you'll change the color and opacity of each circle to make the data more visible against the dark map style. Then, you'll also change the size of each circle to reflect the `DBH`.

### Change color and opacity

Start by importing the correct classes in the `MainActivity.java` file. In this step you'll need the `circleColor` and `circleOpacity` classes.

Then, add the following code to the inside the `public void onStyleLoaded(@NonNull Style style) {...}` method to specify the opacity and color that the layer should use:

{{
  <AndroidTutorialCodeBlock
    filename="MainActivity.java"
    code={snippet.finalJava}
    copyRanges={[
      [15,16],
      [51,53],
      [61,61]
    ]}
  />
}}

Rerun your application, and you will see the same map view with the same point data, but now the circles will be white and semi-transparent.

<div class='align-center'>
<img src='/help/img/android/android-dds-color-opacity.png' alt='initialized map on an Android device' class='wmax360'>
</div>


### Specify radius based on a data attribute

Finally, you'll specify that the radius of circle should be determined by the `DBH` value. Again, you'll start by importing the necessary classes. You'll need the `Function` and `exponential` classes to create a property function, the `Stop` class to establish what the circle radius should be at which `DBH`, and the `circleRadius` class to affect the circleRadius paint property.

Then, replace the code used to specify paint properties for the layer, inside `circleLayer.withProperties( ... )`, to use an exponential property function:

{{
  <AndroidTutorialCodeBlock
    filename="MainActivity.java"
    code={snippet.finalJava}
    copyRanges={[
      [17,21],
      [54,60]
    ]}
  />
}}

Rerun your application, and you will see the map with your data styled in a way that the radius of each circle is determined by the `DBH` value for that point.

<div class='align-center'>
<img src='/help/img/android/android-dds-style-by-attribute.png' alt='map with data styled by attribute on an Android device' class='inline wmax360'>
</div>

## Finished product

You've created a data visualization that illustrates the location and size of street trees all over Washington DC.

<div class='align-center'>
<img src='/help/img/android/android-dds-final-product.gif' alt='map with data styled by attribute on an Android device' class='inline wmax360'>
</div>

{{
  <AndroidTutorialCodeBlock
    filename="MainActivity.java"
    code={snippet.finalJava}
  />
}}

## Next steps

There are many possibilities when using data-driven styling to create beautiful and informative data visualizations for Android applications. Read more about data-driven styling more generally in our [Map design](/help/how-mapbox-works/map-design/) guide and dig into some data-driven styling examples specifically for Android:

- [Style circles categorically](https://github.com/mapbox/mapbox-android-demo/blob/master/MapboxAndroidDemo/src/main/java/com/mapbox/mapboxandroiddemo/examples/dds/StyleCirclesCategoricallyActivity.java): change the color of circles in a circle layer based on a data property.
- [Update by zoom level](https://github.com/mapbox/mapbox-android-demo/blob/master/MapboxAndroidDemo/src/main/java/com/mapbox/mapboxandroiddemo/examples/dds/ChoroplethZoomChangeActivity.java): display state or county population depending on zoom level.
