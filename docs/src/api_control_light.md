# 灯光类模块

**灯光类模块支持搜索和控制的类型**

调光回路（00）
开关回路（01）
混合调光类（09）
混合开关类（10）

### 一、查询状态

**HDLCommand.getLightDeviceStateFromNetwork(AppliancesInfo info)**

**接口描述**

调用该接口，可以查询当前灯光设备的亮度状态信息。

**代码范例**

```java
    //请求获取当前灯光模块状态信息
    HDLCommand.getLightDeviceStateFromNetwork(appliancesInfo);
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
                case HDLApConfig.TYPE_LIGHT_DIMMER:
                case HDLApConfig.TYPE_LIGHT_RELAY:
                case HDLApConfig.TYPE_LIGHT_MIX_DIMMER:
                case HDLApConfig.TYPE_LIGHT_MIX_RELAY:
                    if (appliancesInfo.getChannelNum() == event.getAppliancesInfo().getChannelNum()) {
                        if (!event.isSuccess()) {
                            showToast("获取灯光状态失败，请重新再试");
                            return;
                        }
                        int brightness =  HDLUtlis.getIntegerByObject(event.getAppliancesInfo().getCurState());
                        lightBtn.setText("亮度 = " + brightness);
                        lightText.setText("当前亮度 = " + brightness);
                        showToast("获取状态返回：亮度 = " +  brightness);
                    }
                    break;
                default:
                    //不处理
                    break;
            }
        }
    }
```


### 二、控制改变状态

**HDLCommand.lightCtrl(AppliancesInfo info, int state)**

**接口描述**

调用该接口，可以控制改变灯光模块的，开关状态和亮度值。

**代码范例**

```java
    //发送控制灯光模块指令 
    //appliancesInfo：为需要控制的的模块info参数，里面保存了当前回路模块的所有参数信息
    //lightState：控制改变的亮度值 0~100
    HDLCommand.lightCtrl(appliancesInfo, lightState);

```

**控制状态回调事件监听**

控制状态超时或成功，都会收到LightFeedBackEvent订阅事件，灯光控制回调都统一用这个事件，如果event.isSuccess()值为false，则代表控制超时，没收到设备回复；

并通过比较前设备和回调状态设备的子网号（getDeviceSubnetID）、设备号（getDeviceDeviceID）和回路号（getChannelNum）是否一致，从而判断该消息事件为当前设备控制状态返回的消息，进行下一步处理，不一致则过滤不处理。

**代码范例**
```java

    /**
     * 灯光控制回调Event
     *
     * @param event
     */
    @Subscribe(threadMode = ThreadMode.MAIN)
    public void onLightFeedBackInfoEventMain(LightFeedBackEvent event) {

        if (event.getLightCtrlBackInfo().getAppliancesInfo().getDeviceDeviceID() == appliancesInfo.getDeviceDeviceID()
                && event.getLightCtrlBackInfo().getAppliancesInfo().getDeviceSubnetID() == appliancesInfo.getDeviceSubnetID()
                && event.getLightCtrlBackInfo().getChannelNum() == appliancesInfo.getChannelNum()
        ) {
            //        先判断是否超时
            if (!event.isSuccess()) {
                showToast("灯光控制超时，请重新再试");
                lightBtn.setText("灯光控制超时，请重新再试");
                return;
            }
            int brightness = event.getLightCtrlBackInfo().getBrightness();
            lightState = brightness == 100 ? 0 : 100;//如果返回100重置状态为0，反之重置状态100
            lightBtn.setText("当前亮度 = " + brightness);
            lightText.setText("当前亮度 = " + brightness);
            /*以下为灯光推送示例代码，可以识别哪个继电器，哪个调光灯，哪个回路，也可用作控制回馈。
            按需求调用*/
            String remarks = event.getLightCtrlBackInfo().getRemarks();//获取返回的灯光备注。如果每个灯光回路备注都唯一，可以直接通过备注判断
            String parentRemarks = event.getLightCtrlBackInfo().getParentRemarks();//获取继电器或调光灯备注。这里可以知道是哪个设备返回的
            int num = event.getLightCtrlBackInfo().getChannelNum();//获取回路号。这里可以获取到这个继电器或调光灯的回路号
            showToast("模块：" + parentRemarks + " 的 " + remarks + " 回路，回路号为：" + num + " 返回" + " 亮度为：" + brightness);
            HDLLog.Log("当前亮度 = " + brightness);
        }
    }
```