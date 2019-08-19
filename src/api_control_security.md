# 安防类模块

**安防类模块支持搜索和控制的类型**

安防模块（0） 


### 一、查询布防状态

**HDLCommand.getSecurityStateFromNetwork(AppliancesInfo info)**

**接口描述**

调用该接口，可以查询当前安防的布防状态信息。

**代码范例**

```java
    //请求获取当前安防模块布防状态信息
    HDLCommand.getSecurityStateFromNetwork(appliancesInfo);
```

**查询状态回调事件监听**

查询状态超时或成功，都会收到DeviceStateEvent订阅事件，SDK所有状态回调都统一用这个事件，如果event.isSuccess()值为false，则代表查询超时，没收到设备回复；

开发者可以在所需要获取状态回调的地方，订阅该事件；
并通过比较前设备和回调状态设备的子网号（getDeviceSubnetID）、设备号（getDeviceDeviceID）、AppliancesInfo里面的设备类型（getDeviceType）和回路号（getChannelNum）是否一致，从而判断该消息事件为当前设备查询状态返回的消息，进行下一步处理，不一致则过滤不处理。



**代码范例**
```java
     /**
     * 查询布防状态 回调Event
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
                case HDLApConfig.TYPE_SECURITY_MODULE:
                    if (appliancesInfo.getChannelNum() == event.getAppliancesInfo().getChannelNum()) {
                        if (!event.isSuccess()) {
                            tv_mesSetText("获取布防状态失败，请重新再试");
                            return;
                        }

                        SecurityArmingStateBackInfo mSecurityArmingStateBackInfo = new SecurityArmingStateBackInfo(event.getAppliancesInfo().getArrCurState());
                        if (mSecurityArmingStateBackInfo == null) {
                            tv_mesSetText("获取布防状态失败，请重新再试");
                            return;
                        }

                        int armingStateBack = mSecurityArmingStateBackInfo.getArmingState();
                        if (armingStateBack == 0) {   //读取返回0为撤防
                            armingState = SecurityParser.ARMING_DISARMING;
                            spinner_arming.setSelection(armingState - 1, true);
                            tv_mesSetText("获取成功，当前状态：" + armingTypeItems[armingState - 1]);
                        } else if (armingStateBack >= 1 && armingStateBack <= 6) {
                            armingState = armingStateBack;//更新状态值
                            spinner_arming.setSelection(armingState - 1, true);
                            tv_mesSetText("获取成功，当前状态：" + armingTypeItems[armingState - 1]);
                        } else {
                            tv_mesSetText("获取成功，未知布防状态");
                        }

                    }
                    break;
            }
        }
    }

```

**注：返回的 armingState布防状态对应关系： 5 = 白天布防 4 = 晚上有客布防 3 = 夜间布防 2 = 离开布防 1 = 假期布防  0 = 撤防**

### 二、控制改变状态

**HDLCommand.securityArmingCtrl(AppliancesInfo info, int state)**

**接口描述**

调用该接口，修改安防模块的布防状态。

**注：控制的state布防状态对应关系：6 = 撤防 5 = 白天布防 4 = 晚上有客布防 3 = 夜间布防 2 = 离开布防 1 = 假期布防**
    **撤防状态下才能布防，所以想修改其他布防模式需先撤防**

**代码范例**

```java
    //发送修改布防状态命令 
    HDLCommand.securityArmingCtrl(appliancesInfo, armingState);

```

**控制状态回调事件监听**

布防超时或成功，都会收到SecurityArmingFeedBackEvent订阅事件，如果event.isSuccess()值为false，则代表控制超时，没收到设备回复；

并通过比较前设备和回调状态设备的子网号（getDeviceSubnetID）、设备号（getDeviceDeviceID）和回路号（getChannelNum）是否一致，从而判断该消息事件为当前设备控制状态返回的消息，进行下一步处理，不一致则过滤不处理。

**代码范例**
```java

 /**
     * 布防设置回调Event
     *
     * @param event
     */
    @Subscribe(threadMode = ThreadMode.MAIN)
    public void onSecurityArmingFeedBackEventMain(SecurityArmingFeedBackEvent event) {
        if (event.getAppliancesInfo().getDeviceSubnetID() == appliancesInfo.getDeviceSubnetID()
                && event.getAppliancesInfo().getDeviceDeviceID() == appliancesInfo.getDeviceDeviceID()
                && event.getAppliancesInfo().getChannelNum() == appliancesInfo.getChannelNum()
        ) {
            //先判断是否超时
            if (!event.isSuccess()) {
                tv_mesSetText("布防设置超时，请重新再试");
                return;
            }

            int armingStateGet = event.getArmingState();
            if (armingStateGet >= 1 && armingStateGet <= 6) {
                armingState = armingStateGet;
                spinner_arming.setSelection(armingState - 1, true);
                tv_mesSetText("布防成功，当前状态：" + armingTypeItems[armingState - 1]);
            } else if (armingState == SecurityParser.ARMING_FAIL) {
                tv_mesSetText("布防失败");
            } else {
                tv_mesSetText("未知布防状态");
            }
        }
    }

```