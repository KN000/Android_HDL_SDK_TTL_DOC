# 通用开关模块

**通用开关模块的类型**

通用开关（大类 100 小类 0）

### 一、查询状态

**接口描述**

HDLCommand.getCommonSwitchStateFromNetwork(AppliancesInfo info)

调用该接口，可以查询当前通用开关设备的开关状态。

**代码范例**

```java
    //请求获取通用开关状态信息
    HDLCommand.getCommonSwitchStateFromNetwork(appliancesInfo);
```

**查询状态回调事件监听**

查询状态超时或成功，都会收到onCommonSwitchStateEventMain订阅事件，如果event.isSuccess()值为false，则代表查询超时，没收到设备回复；

开发者可以在所需要获取状态回调的地方，订阅该事件；
并通过比较前设备和回调状态设备的子网号（getDeviceSubnetID）、设备号（getDeviceDeviceID）、AppliancesInfo里面的设备类型（getDeviceType）和回路号（getChannelNum）是否一致，从而判断该消息事件为当前设备查询状态返回的消息，进行下一步处理，不一致则过滤不处理。



**代码范例**
```java
    /**
     * 通用开关状态 回复回调Event
     *
     * @param event
     */
    @Subscribe(threadMode = ThreadMode.MAIN)
    public void onCommonSwitchStateEventMain(CommonSwitchStateBackEvent event) {
        if (event.getCommonSwitchBackInfo().getAppliancesInfo().getDeviceDeviceID() == appliancesInfo.getDeviceDeviceID()
                && event.getCommonSwitchBackInfo().getAppliancesInfo().getDeviceSubnetID() == appliancesInfo.getDeviceSubnetID()
                && event.getCommonSwitchBackInfo().getAppliancesInfo().getChannelNum() == appliancesInfo.getChannelNum()
        ) {
            if (!event.isSuccess()) {
                showToast("通用开关读取状态超时，请重新再试");
                return;
            }

//            showToast("通用开关控制成功");

            switchState =  event.getCommonSwitchBackInfo().getSwitchState();
            if(switchState>0){
                switchText.setText("当前状态：开");
            }else{
                switchText.setText("当前状态：关");
            }
        }

    }
```


### 二、控制改变状态

**接口描述**

HDLCommand.commonSwitchCtrl(AppliancesInfo info, int )

调用该接口，可以控制通用开关模块的开关状态。

**代码范例**

```java
    //发送控制通用开关指令 
    //appliancesInfo：为需要控制的的模块info参数，里面保存了当前回路模块的所有参数信息
    //0=关 255=开
    HDLCommand.commonSwitchCtrl(appliancesInfo, state);

```

**控制状态回调事件监听**

控制超时或成功，都会收到CommonSwitchCtrlBackEvent订阅事件，通用开关控制回调都统一用这个事件，如果event.isSuccess()值为false，则代表控制超时，没收到设备回复；

并通过比较前设备和回调状态设备的子网号（getDeviceSubnetID）、设备号（getDeviceDeviceID）和回路号（getChannelNum）是否一致，从而判断该消息事件为当前设备控制状态返回的消息，进行下一步处理，不一致则过滤不处理。

**代码范例**
```java
    /**
     * 通用开关控制回调Event
     *
     * @param event
     */
    @Subscribe(threadMode = ThreadMode.MAIN)
    public void onCommonSwitchCtrlEventMain(CommonSwitchCtrlBackEvent event) {
        if (event.getCommonSwitchBackInfo().getAppliancesInfo().getDeviceDeviceID() == appliancesInfo.getDeviceDeviceID()
                && event.getCommonSwitchBackInfo().getAppliancesInfo().getDeviceSubnetID() == appliancesInfo.getDeviceSubnetID()
                && event.getCommonSwitchBackInfo().getAppliancesInfo().getChannelNum() == appliancesInfo.getChannelNum()
        ) {
            if (!event.isSuccess()) {
                showToast("通用开关控制超时，请重新再试");
                return;
            }

            showToast("通用开关控制成功");

            switchState =  event.getCommonSwitchBackInfo().getSwitchState();
            if(switchState>0){
                 switchText.setText("当前状态：开");
            }else{
                switchText.setText("当前状态：关");
            }
        }

    }
```