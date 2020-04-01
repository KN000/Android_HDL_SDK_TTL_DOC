# 场景控制（逻辑模块）

**场景控制的类型**
大类
逻辑模块（12）
全局逻辑模块（17）

### 一、查询状态

**接口描述**

HDLCommand.getSceneNumStateFromNetwork(AppliancesInfo info)

调用该接口，可以查询当前场景号信息。

**代码范例**

```java
    //请求获取通用开关状态信息
    HDLCommand.getSceneNumStateFromNetwork(appliancesInfo);
```

**查询状态回调事件监听**

    查询状态超时或成功，都会收到LogicFeedBackEvent订阅事件，如果event.isSuccess()值为false，则代表查询超时，没收到设备回复；
   同时还会收到ScenesStateBackEvent订阅事件，收到场景变化通知或者控制回复都会推送该事件，可以根据该事件 判断当前场景的状态信息

**代码范例**
```java
   
    /**
     * 当前场景状态 通知Event
     *
     * @param event
     */
    @Subscribe(threadMode = ThreadMode.MAIN)
    public void onScenesStateBackEventMain(ScenesStateBackEvent event) {
        if (event.getDeviceDeviceID() == appliancesInfo.getDeviceDeviceID()
                && event.getDeviceSubnetID() == appliancesInfo.getDeviceSubnetID()) {
            //相同区号相同场景号，其他场景号、则为执行了其他场景号
            int mAreaNum = event.getAreaNum();
            int mSceneNum = event.getSceneNum();

            //可以自己通过区号 和 场景号 来匹配判断当前执行了什么场景
            sceneText.setText("当前模块执行的场景区号: " + mAreaNum + "  场景号: "+mSceneNum);

        }
    }
```


### 二、控制改变状态

**接口描述**

HDLCommand.commonSwitchCtrl(AppliancesInfo info, int )

调用该接口，可以控制通用开关模块的开关状态。

**代码范例**

```java
    //发送场景控制指令 
    //appliancesInfo：为需要控制的的模块info参数，里面保存了当前回路模块的所有参数信息
    HDLCommand.logicCtrl(appliancesInfo);

```

**控制状态回调事件监听**

控制状态超时或成功，都会收到LogicFeedBackEvent 订阅事件，通用开关控制回调都统一用这个事件，如果event.isSuccess()值为false，则代表控制超时，没收到设备回复；


**代码范例**
```java
      /**
     * 逻辑模块控制回调Event
     *
     * @param event
     */
    @Subscribe(threadMode = ThreadMode.MAIN)
    public void onLogicFeedBackInfoEventMain(LogicFeedBackEvent event) {
//        先判断是否超时
        if (event.getLogicCtrlBackInfo().getAppliancesInfo().getDeviceDeviceID() == appliancesInfo.getDeviceDeviceID()
                && event.getLogicCtrlBackInfo().getAppliancesInfo().getDeviceSubnetID() == appliancesInfo.getDeviceSubnetID()
                && event.getLogicCtrlBackInfo().getAppliancesInfo().getChannelNum() == appliancesInfo.getChannelNum()
        ) {
            if (!event.isSuccess()) {
                showToast("场景控制超时，请重新再试");
                return;
            }
            showToast("场景控制成功");
        }
    }

```