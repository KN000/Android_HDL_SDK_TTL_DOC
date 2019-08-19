# 控制设备

### 一、控制设备

SDK所有的设备和Mcu控制方法在HDLCommand类中，

开发者可以通过调用里面的方法，并传入对应控制设备的AppliancesInfo参数，则可以成功发送控制指令。

**代码范例**

```java
    //发送控制灯光模块指令 
    //appliancesInfo：为需要控制的的模块info参数，里面保存了当前回路模块的所有参数信息
    //lightState：控制改变的亮度值 0~100
    HDLCommand.lightCtrl(appliancesInfo, lightState);
```

### 二、设备类型说明

通过获取设备appliancesInfo里面的bigType参数，可以判断当前设备的类型，跳转加载对应的控制页面，并发送对应的控制命令。

每个bigType大类区分哪个大类模块后，可以再根据DeviceType设备类型来细分具体是哪种设备，例如下面代码范例HVAC空调模块。

**代码范例**

```java
switch (bigType) {
    case Configuration.LIGTH_BIG_TYPE:
        //灯光模块
        break;
    case Configuration.CURTAIN_BIG_TYPE:
        //窗帘模块
        break;
    case Configuration.AIR_BIG_TYPE:
        //空调模块
        if(appliancesInfos.get(position).getDeviceType() == HDLApConfig.TYPE_AC_HVAC){
            //HDLApConfig.TYPE_AC_HVAC为HVAC空调模块
        }else {
            //其他空调模块    
        }
        break;
    case Configuration.LOGIC_BIG_TYPE:
        //逻辑模块
        break;
    case Configuration.AUDIO_BIG_TYPE:
        //音乐模块
        break;
    case Configuration.SECURITY_BIG_TYPE:
        //安防模块
        break;

    default:
        break;
}
```


### 三、AppliancesInfo类说明

**类描述**

AppliancesInfo类是SDK搜索或者手机添加设备后，用来保存当前回路模块的所有信息。


**类参数说明代码范例**

```java
    .......
    private String deviceName;//设备名称
    private int bigType;//大类
    private int littleType;//小类
    private int channelNum;//回路号
    private int deviceSubnetID;//设备子网号
    private int deviceDeviceID;//设备号
    private String remarks;//备注
    private String parentRemarks;//模块备注
    private int deviceType;//设备类型
    .......
```

每个模块都会有自己的 大类、小类、子网号、设备号、回路号、设备类型、备注等参数，根据这些参数可以识别出该设备是哪类设备，在哪个子网号、设备号、以及回路号下面，用于区分不同回路设备。后面对设备控制获取状态，监听回复都需要用到里面的这些参数。

