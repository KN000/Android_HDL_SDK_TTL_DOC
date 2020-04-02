# 场景控制（逻辑模块）

**场景控制的类型**

逻辑模块（大类 12 小类 0 ）
全局逻辑模块（大类 17 小类 0） 

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

    查询状态成功，会收到ScenesStateBackEvent订阅事件，同时场景变化也会收到该事件回调

**代码范例**
```java
   
      /**
     * 当前场景状态变化都会推送这个通知Event
     * 更加项目需要， 这里的区号有的情况下不需处理，有的设备固定是1 只需处理场景号，判断目前执行了那个场景就行
     * @param event
     */
    @Subscribe(threadMode = ThreadMode.MAIN)
    public void onScenesStateBackEventMain(ScenesStateBackEvent event) {
        if (event.getDeviceDeviceID() == appliancesInfo.getDeviceDeviceID()
                && event.getDeviceSubnetID() == appliancesInfo.getDeviceSubnetID()) {
//            //
//            int mAreaNum = event.getAreaNum();
//            int mSceneNum = event.getSceneNum();
//
//            //可以自己通过区号 和 场景号 来匹配判断当前执行了什么场景
//            sceneText.setText("当前模块执行的场景区号: " + mAreaNum + "  场景号: "+mSceneNum);
            
            int mSceneNum = event.getSceneNum();
            //可以自己通过场景号 来匹配判断当前执行了什么场景
            sceneText.setText("当前模块执行的场景号: "+mSceneNum);

        }
    }

```


### 二、控制改变状态

**接口描述**

HDLCommand.logicCtrl(AppliancesInfo info)

调用该接口，可以执行相应的场景。

**代码范例**

```java
    //发送场景控制指令 
    //appliancesInfo：为需要控制的的模块info参数，里面保存了当前回路模块的所有参数信息
    HDLCommand.logicCtrl(appliancesInfo);

```

**控制状态回调事件监听**

控制场景超时或成功，都会收到LogicFeedBackEvent 订阅事件，通用开关控制回调都统一用这个事件，如果event.isSuccess()值为false，则代表控制超时，没收到设备回复；

如果控制成功的话，场景状态变化，也会收到ScenesStateBackEvent订阅事件，可以自己通过场景号 来匹配判断当前执行了什么场景


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

   /**
     * 当前场景状态变化都会推送这个通知Event
     * 更加项目需要， 这里的区号有的情况下不需处理，有的设备固定是1 只需处理场景号，判断目前执行了那个场景就行
     * @param event
     */
    @Subscribe(threadMode = ThreadMode.MAIN)
    public void onScenesStateBackEventMain(ScenesStateBackEvent event) {
        if (event.getDeviceDeviceID() == appliancesInfo.getDeviceDeviceID()
                && event.getDeviceSubnetID() == appliancesInfo.getDeviceSubnetID()) {
//            //
//            int mAreaNum = event.getAreaNum();
//            int mSceneNum = event.getSceneNum();
//
//            //可以自己通过区号 和 场景号 来匹配判断当前执行了什么场景
//            sceneText.setText("当前模块执行的场景区号: " + mAreaNum + "  场景号: "+mSceneNum);
            
            int mSceneNum = event.getSceneNum();
            //可以自己通过场景号 来匹配判断当前执行了什么场景
            sceneText.setText("当前模块执行的场景号: "+mSceneNum);

        }
    }

```