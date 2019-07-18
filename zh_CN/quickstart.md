# Android 快速入门
<h4>1. 添加依赖库</h4>
```java
复制 facelibrary-release.aar和openCVLibrary331-release.aar 到工程下 app/libs 目录
```
<h4>2. 添加编译 aar 信息</h4>
```java
compile(name: 'facelibrary-release', ext: 'aar')
compile(name: 'openCVLibrary331-release', ext: 'aar')
    
到 build.gradle 的 dependencies
```
<h4>3. 设置权限</h4>
```java
在AndroidManifest.xml文件中添加如下权限

<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
<uses-permission android:name="android.permission.CAMERA"/>
<uses-permission android:name="android.permission.INTERNET"/>
<uses-feature android:name="android.hardware.camera" android:required="true"/>
```
<h4>4. 在程序中定义 FaceAPP 变量</h4>
```java
private FaceAPP face = FaceAPP.GetInstance(); //face 作为成员变量
```
<h4>5. 通过 face 调用 Android API 接口</h4>


