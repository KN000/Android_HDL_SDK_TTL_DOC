# 空调类模块

**空调类模块支持搜索和控制的类型**

HVAC 模块（0）
通用空调面板（3）

### 一、HVAC模块查询状态

**接口描述**

HDLCommand.getHVACDeviceStateFromNetwork(AppliancesInfo info)

调用该接口，获取HVAC空调设备状态。

注：HVAC空调DeviceType设备类型为HDLApConfig.TYPE_AC_HVAC。

**代码范例**

```java
    //查询HVAC空调模块状态信息
    HDLCommand.getHVACDeviceStateFromNetwork(appliancesInfo);
```

**查询状态回调事件监听**

查询状态超时或成功，都会收到DeviceStateEvent订阅事件，SDK所有状态回调都统一用这个事件，如果event.isSuccess()值为false，则代表查询超时，没收到设备回复；

开发者可以在所需要获取状态回调的地方，订阅该事件；
并通过比较前设备和回调状态设备的子网号（getDeviceSubnetID）、设备号（getDeviceDeviceID）、AppliancesInfo里面的设备类型（getDeviceType）和回路号（getChannelNum）是否一致，从而判断该消息事件为当前设备查询状态返回的消息，进行下一步处理，不一致则过滤不处理。



**代码范例**
```java
    /**
     * 获取单一设备状态回调Event
     *
     * @param event
     */
    @Subscribe(threadMode = ThreadMode.MAIN)
    public void onDeviceStateEventMain(DeviceStateEvent event) {
        if (event.getAppliancesInfo().getDeviceSubnetID() == appliancesInfo.getDeviceSubnetID()
                && event.getAppliancesInfo().getDeviceDeviceID() == appliancesInfo.getDeviceDeviceID()
        ) {
            //这个返回的信息是当前状态的
            switch (event.getAppliancesInfo().getDeviceType()) {
                case HDLApConfig.TYPE_AC_HVAC:
                    if (appliancesInfo.getChannelNum() == event.getAppliancesInfo().getChannelNum()) {
                        if (!event.isSuccess()) {
                            showToast("获取空调状态失败，请重新再试");
                            return;
                        }
                        AirHVACBackInfo mAirHVACBackInfo = new AirHVACBackInfo(event.getAppliancesInfo());
                        if (mAirHVACBackInfo == null) {
                            showToast("获取空调状态失败，请重新再试");
                            return;
                        }
                        showAirHVACBackInfo(mAirHVACBackInfo);
                    }
                    break;
                default:
                    //不处理
                    break;
            }
        }
    }

```


### 二、HVAC模块控制改变状态

**接口描述**

HDLCommand.airCtrl(AppliancesInfo info, int type, int state)

type：需要控制功能命令参数

state：需要控制功能对应的状态值

调用该接口，可以控制改变HVAC空调模块的，开、光、模式、风速、温度等状态。

模式：制冷、制热、通风、自动、抽湿5种模式。

风速：自动、高风、中风、低风

温度：温度范围16~30

**代码范例**

```java
    .......
    //发送控制空调模块打开指令 
    HDLCommand.airCtrl(appliancesInfo, AirCtrlParser.airSwich, AirCtrlParser.airOn);//空调开
    .......

    .......
    //发送控制空调模块改变模式指令 
    HDLCommand.airCtrl(appliancesInfo, AirCtrlParser.airMode, AirCtrlParser.airModeAuto);//空调模式自动
    .......

    .......
    //发送控制空调模块改变风速指令 
    HDLCommand.airCtrl(appliancesInfo, AirCtrlParser.airSpeed, AirCtrlParser.airSpeedHigh);//风速高风
    .......

    .......
    //发送控制空调模块改变温度指令 
    HDLCommand.airCtrl(appliancesInfo, AirCtrlParser.refTem, tempInt);//制冷温度
    .......

```

**控制状态回调事件监听**

控制状态超时或成功，都会收到AirHVACFeedBackEvent订阅事件，如果event.isSuccess()值为false，则代表控制超时，没收到设备回复；

并通过比较前设备和回调状态设备的子网号（getDeviceSubnetID）、设备号（getDeviceDeviceID）和回路号（getChannelNum）是否一致，从而判断该消息事件为当前设备控制状态返回的消息，进行下一步处理，不一致则过滤不处理。

**代码范例**
```java
    /**
     * 空调模块控制回调Event
     *
     * @param event
     */
    @Subscribe(threadMode = ThreadMode.MAIN)
    public void onAirHVACFeedBackEventMain(AirHVACFeedBackEvent event) {
        if (event.getAirHVACBackInfo().getAppliancesInfo().getDeviceDeviceID() == appliancesInfo.getDeviceDeviceID()
                && event.getAirHVACBackInfo().getAppliancesInfo().getDeviceSubnetID() == appliancesInfo.getDeviceSubnetID()
                && event.getAirHVACBackInfo().getAppliancesInfo().getChannelNum() == appliancesInfo.getChannelNum()
        ) {
            //        先判断是否超时
            if (!event.isSuccess()) {
                showToast("空调控制超时，请重新再试");
                return;
            }
            AirHVACBackInfo mAirHVACBackInfo = event.getAirHVACBackInfo();
            showAirHVACBackInfo(mAirHVACBackInfo);
        }
    }

    .......
    private void showAirHVACBackInfo(AirHVACBackInfo mAirHVACBackInfo){
        String message = "";
        if(mAirHVACBackInfo.getIsOn() == AirCtrlParser.airOn) {
            message = getSwichStateString(mAirHVACBackInfo.getIsOn());
            message += "\n" + getModeStateString(mAirHVACBackInfo.getAirMode());//模式
            message += "\n" + getSpeedStateString(mAirHVACBackInfo.getAirSpeed());//风速
            message += "\n制冷模式温度：" + mAirHVACBackInfo.getRefTemp();
            message += "\n制热模式温度：" + mAirHVACBackInfo.getHeatTemp();
            message += "\n自动模式温度：" + mAirHVACBackInfo.getAutoTemp();
            message += "\n抽湿模式温度：" + mAirHVACBackInfo.getWettedTemp();
        }else {
            message = getSwichStateString(mAirHVACBackInfo.getIsOn());
        }
        airText.setText(message);
        showToast(message);
        HDLLog.Log(message);

    }

```



### 三、通用空调面板查询状态

**接口描述**

通用空调面板不支持查询状态，

注：通用空调面板空调DeviceType设备类型为HDLApConfig.TYPE_AC_PANEL。


**状态改变回调事件监听**

空调面板状态改变会收到DeviceStateEvent订阅事件，SDK所有状态回调都统一用这个事件，如果event.isSuccess()值为false，则代表查询超时，没收到设备回复；

开发者可以在所需要获取状态改变的地方，订阅该事件；
并通过比较前设备和回调状态设备的子网号（getDeviceSubnetID）、设备号（getDeviceDeviceID）、AppliancesInfo里面的设备类型（getDeviceType）和回路号（getChannelNum）是否一致，从而判断该消息事件为当前设备查询状态返回的消息，进行下一步处理，不一致则过滤不处理。


**代码范例**

```java

    /**
     * 获取单一设备状态回调Event
     *
     * @param event
     */
    @Subscribe(threadMode = ThreadMode.MAIN)
    public void onDeviceStateEventMain(DeviceStateEvent event) {
        if (event.getAppliancesInfo().getDeviceSubnetID() == appliancesInfo.getDeviceSubnetID()
                && event.getAppliancesInfo().getDeviceDeviceID() == appliancesInfo.getDeviceDeviceID()
        ) {
            //这个返回的信息是当前状态的
            switch (event.getAppliancesInfo().getDeviceType()) {
//                case HDLApConfig.TYPE_AC_HVAC:
                case HDLApConfig.TYPE_AC_PANEL:
                    if (appliancesInfo.getChannelNum() == event.getAppliancesInfo().getChannelNum()) {
                        if (!event.isSuccess()) {
                            showToast("获取空调状态失败，请重新再试");
                            return;
                        }

                        byte[] curState = event.getAppliancesInfo().getArrCurState();
                        switch (curState[0] & 0xFF) {
                            case AirCtrlParser.airSwich:
                                //是空调开关状态返回
                                break;

                            case AirCtrlParser.airSpeed:
                                //是风速状态返回
                                break;
                            case AirCtrlParser.airMode:
                                //是空调模式状态返回
                                break;
                            case AirCtrlParser.refTem:
                                //是空调制冷温度状态返回
                                break;
                            case AirCtrlParser.heatTem:
                                //是空调制热温度状态返回
                                break;
                            case AirCtrlParser.autoTem:
                                //是空调自动温度状态返回
                                break;
                            case AirCtrlParser.dehumTem:
                                //是空调抽湿温度状态返回
                                break;
                            case AirCtrlParser.upTem:
                                airTempState = curState[1] & 0xFF;
                                airText.setText("空调调温，上升温度：" + (curState[1] & 0xFF));
                                showToast("空调调温，上升温度：" + (curState[1] & 0xFF));
                                HDLLog.Log("空调调温，上升温度：" + (curState[1] & 0xFF));
                                break;
                            case AirCtrlParser.downTem:
                                airTempState = curState[1] & 0xFF;
                                airText.setText("空调调温，下降温度：" + (curState[1] & 0xFF));
                                showToast("空调调温，下降温度：" + (curState[1] & 0xFF));
                                HDLLog.Log("空调调温，下降温度：" + (curState[1] & 0xFF));
                                break;
                        }
                    }
                    break;
                default:
                    //不处理
                    break;
            }
        }
    }



```

### 四、通用空调面板控制改变状态

**接口描述**

HDLCommand.airCtrl(AppliancesInfo info, int type, int state)

type：需要控制功能命令参数

state：需要控制功能对应的状态值

调用该接口，可以控制改变通用空调面板模块的，开、光、模式、风速、温度等状态。

模式：制冷、制热、通风、自动、抽湿5种模式。

风速：自动、高风、中风、低风

温度：温度范围16~30

注：通用空调面板和HVAC空调模块不一样，通用空调面板控制什么状态就返回什么状态。

例如：控制开关状态，空调面板只会回复当前开关状态，模式、风速、温度等其他状态不会返回，所以监听回调方法时需先判断，当前回复的是什么状态参数，再进行下一步处理。

**代码范例**

```java
    .......
    //发送控制空调模块打开指令 
    HDLCommand.airCtrl(appliancesInfo, AirCtrlParser.airSwich, AirCtrlParser.airOn);//空调开
    .......

    .......
    //发送控制空调模块改变模式指令 
    HDLCommand.airCtrl(appliancesInfo, AirCtrlParser.airMode, AirCtrlParser.airModeAuto);//空调模式自动
    .......

    .......
    //发送控制空调模块改变风速指令 
    HDLCommand.airCtrl(appliancesInfo, AirCtrlParser.airSpeed, AirCtrlParser.airSpeedHigh);//风速高风
    .......

    .......
    //发送控制空调模块改变温度指令 
    HDLCommand.airCtrl(appliancesInfo, AirCtrlParser.refTem, tempInt);//制冷温度
    .......

```

**控制状态回调事件监听**

控制状态超时或成功，都会收到AirFeedBackEvent订阅事件，如果event.isSuccess()值为false，则代表控制超时，没收到设备回复；

并通过比较前设备和回调状态设备的子网号（getDeviceSubnetID）、设备号（getDeviceDeviceID）和回路号（getChannelNum）是否一致，从而判断该消息事件为当前设备控制状态返回的消息，进行下一步处理，不一致则过滤不处理。


**代码范例**
```java
   /**
     * 空调模块控制回调Event
     *
     * @param event
     */
    @Subscribe(threadMode = ThreadMode.MAIN)
    public void onAirFeedBackInfoEventMain(AirFeedBackEvent event) {
        if (event.getAirCtrlBackInfo().getAppliancesInfo().getDeviceDeviceID() == appliancesInfo.getDeviceDeviceID()
                && event.getAirCtrlBackInfo().getAppliancesInfo().getDeviceSubnetID() == appliancesInfo.getDeviceSubnetID()
                && event.getAirCtrlBackInfo().getAppliancesInfo().getChannelNum() == appliancesInfo.getChannelNum()
        ) {
            //        先判断是否超时
            if (!event.isSuccess()) {
                showToast("空调控制超时，请重新再试");
                return;
            }

            byte[] curState = event.getAirCtrlBackInfo().getCurState();
            switch (curState[0] & 0xFF) {
                case AirCtrlParser.airSwich:
                    //是空调开关状态返回
                    break;
                case AirCtrlParser.airSpeed:
                    //是风速状态返回
                    break;
                case AirCtrlParser.airMode:
                    //是空调模式状态返回
                    break;
                case AirCtrlParser.refTem:
                    //是空调制冷温度状态返回
                    break;
                case AirCtrlParser.heatTem:
                    //是空调制热温度返回
                    break;
                case AirCtrlParser.autoTem:
                    //是空调自动温度返回
                    break;
                case AirCtrlParser.dehumTem:
                    //是空调抽湿热温度返回
                    break;
                case AirCtrlParser.upTem:
                    airTempState = curState[1] & 0xFF;
                    airText.setText("空调调温，上升温度：" + (curState[1] & 0xFF));
                    showToast("空调调温，上升温度：" + (curState[1] & 0xFF));
                    HDLLog.Log("空调调温，上升温度：" + (curState[1] & 0xFF));
                    break;
                case AirCtrlParser.downTem:
                    airTempState = curState[1] & 0xFF;
                    airText.setText("空调调温，下降温度：" + (curState[1] & 0xFF));
                    showToast("空调调温，下降温度：" + (curState[1] & 0xFF));
                    HDLLog.Log("空调调温，下降温度：" + (curState[1] & 0xFF));
                    break;

            }
        }
    }
```