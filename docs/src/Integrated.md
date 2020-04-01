# 集成SDK
### 一、创建工程

在Android Studio中建立你的工程。

### 二、build.gradle 配置

将SDK包中的aar文件( 如HDL_TTLSDK_V1.0.1_r.arr )导入到现有的工程中，放入到libs目录下。


build.gradle 文件里添加集成准备中下载的dependencies 依赖库。

```groovy

    dependencies {
        implementation fileTree(include: ['*.jar','*.aar'], dir: 'libs')
        implementation 'org.greenrobot:eventbus:3.0.0'
    }
```

### 三、Application中初始化SDK。

**代码范例**

```java
public class HDLApplication extends Application {
    @Override
    public void onCreate() {
        super.onCreate();
        //HDL_PATH_NAME (串口路径)
        //HDL_BAUDRATE  (波特率)，波特率默认115200
        HDLTtlSdk.init(this, HDL_PATH_NAME, HDL_BAUDRATE);
    }
}
```

### 四、日志打印开关

SKD打印日志默认是打开，用户可以配置为false，即可关闭SDK打印日志。

**代码范例**

```java
public class HDLApplication extends Application {
    @Override
    public void onCreate() {
        super.onCreate();
        HDLTtlSdk.init(this, HDL_PATH_NAME, HDL_BAUDRATE);
        HDLTtlSdk.setHDLLogOpen(false);//关闭SDK日志打印
    }
}
```

### 五、EventBus 使用

SKD的大部分回调事件，通过EventBus发布回调事件，所以开发者项目中需要集成EventBus，并在需要的Activity中执行注册订阅事件（register）和取消事件订阅（unregister）。

**代码范例**

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    initEventBusRegister();//注册订阅事件

}
/**
 * 注册事件分发初始化
 */
private void initEventBusRegister() {
    if (isRegisterEventBus()) {
        if (!EventBus.getDefault().isRegistered(this)) {
             EventBus.getDefault().register(this);
        }
    }
}

@Override
protected void onDestroy() {
    super.onDestroy();
    unEventBusRegister();//取消事件订阅
}

/**
* 注册事件分发反始化
 */
private void unEventBusRegister() {
    if (isRegisterEventBus()) {
        EventBus.getDefault().unregister(this);
    }
}
```