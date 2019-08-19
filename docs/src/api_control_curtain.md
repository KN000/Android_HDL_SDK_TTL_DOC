# 窗帘类模块

**窗帘类模块支持搜索和控制的类型**

开合帘电机（00）
卷帘电机（01）
窗帘模块（02） 

### 一、查询状态

**HDLCommand.getCurtainDeviceStateFromNetwork(AppliancesInfo info)**

**接口描述**

调用该接口，可以查询当前窗帘类设备的开关状态信息。

**代码范例**

```java
    //请求获取当前窗帘模块状态信息
    HDLCommand.getCurtainDeviceStateFromNetwork(appliancesInfo);
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
                case HDLApConfig.TYPE_CURTAIN_GLYSTRO:
                case HDLApConfig.TYPE_CURTAIN_ROLLER:
                case HDLApConfig.TYPE_CURTAIN_MODULE:
                    if (appliancesInfo.getChannelNum() == event.getAppliancesInfo().getChannelNum()) {
                        if (!event.isSuccess()) {
                            showToast("获取窗帘状态失败，请重新再试");
                            return;
                        }
                        showMessage = "";
                        //窗帘模块：curState:0=停止,1=打开,2=关闭。
                        //开合帘电机，卷帘电机：curState:1-100开合度。
                        int curState = HDLUtlis.getIntegerByObject(event.getAppliancesInfo().getCurState());
                        if (event.getAppliancesInfo().getDeviceType() == HDLApConfig.TYPE_CURTAIN_MODULE) {//判断是否为窗帘模块,否则为开合帘或卷帘电机
                            switch (curState) {
                                case CurtainCtrlParser.curtainOff:
                                    showMessage = "窗帘关";
                                    curtainBtn.setText(showMessage);
                                    curText1.setText(showMessage);
                                    curtainState = curState;
                                    break;
                                case CurtainCtrlParser.curtainOn:
                                    showMessage = "窗帘开";
                                    curtainBtn.setText(showMessage);
                                    curText1.setText(showMessage);
                                    curtainState = curState;
                                    break;
                                case CurtainCtrlParser.curtainPause:
                                    showMessage = "窗帘暂停";
                                    curtainBtn.setText(showMessage);
                                    curText1.setText(showMessage);
                                    curtainState = curState;
                                    break;
                            }
                        } else {
                            showMessage = "窗帘开到" + curState + "%";
                            curtainBtn5.setText(showMessage);
                            curtainState = curState;
                        }

                        showToast("获取成功：" + showMessage);
                    }
                    break;
                default:
                    //不处理
                    break;
            }
        }
    }

```

**备注**

HDLApConfig.TYPE_CURTAIN_MODULE 为窗帘模块，该类型模块，只有开、关、停止 3种状态；

HDLApConfig.TYPE_CURTAIN_GLYSTRO 为开合帘电机，该类型模块，返回状态0-100%，表示打开百分比；

HDLApConfig.TYPE_CURTAIN_ROLLER 为卷帘电机，该类型模块，返回状态0-100%，表示打开百分比；



### 二、控制改变状态

**HDLCommand.curtainCtrl(AppliancesInfo info, int state)**

**接口描述**

调用该接口，可以控制改变窗帘类模块的，打开、关闭、停止和打开百分比（窗帘模块不能控制百分百，开合帘电机、卷帘电机才能）。

**代码范例**

```java
    ........
    //发送控制窗帘打开命令
    HDLCommand.curtainCtrl(appliancesInfo, CurtainCtrlParser.curtainOn);
    ........

    ........
    //发送控制开合帘电机打开到50%
    HDLCommand.curtainCtrl(appliancesInfo, 50);
    ........

```

**控制状态回调事件监听**

控制状态超时或成功，都会收到CurtainFeedBackEvent订阅事件，窗帘类控制回调都统一用这个事件，如果event.isSuccess()值为false，则代表控制超时，没收到设备回复；

并通过比较前设备和回调状态设备的子网号（getDeviceSubnetID）、设备号（getDeviceDeviceID）和回路号（getChannelNum）是否一致，从而判断该消息事件为当前设备控制状态返回的消息，进行下一步处理，不一致则过滤不处理。

**代码范例**
```java

      /**
     * 窗帘模块控制回调Event
     *
     * @param event
     */
    @Subscribe(threadMode = ThreadMode.MAIN)
    public void onCurtainFeedBackInfoEventMain(CurtainFeedBackEvent event) {
//        先判断是否超时
        HDLLog.Log("onCurtainFeedBackInfoEventMain in");
        if (event.getCurtainCtrlBackInfo().getAppliancesInfo().getDeviceSubnetID() == appliancesInfo.getDeviceSubnetID()
                && event.getCurtainCtrlBackInfo().getAppliancesInfo().getDeviceDeviceID() == appliancesInfo.getDeviceDeviceID()
                && event.getCurtainCtrlBackInfo().getNum() == appliancesInfo.getChannelNum()
        ) {

            if (!event.isSuccess()) {
                showToast("窗帘控制超时，请重新再试");
                return;
            }

            int curState = event.getCurtainCtrlBackInfo().getState();
            //窗帘模块：curState:0=停止,1=打开,2=关闭。
            //开合帘电机，卷帘电机：curState:1-100开合度。也会返回0，1，2的状态
            //建议开合帘电机，卷帘电机按停止后再读取当前状态来获取当前状态值

            String remarks = event.getCurtainCtrlBackInfo().getRemarks();
            String parentRemarks = event.getCurtainCtrlBackInfo().getParentRemarks();
            int num = event.getCurtainCtrlBackInfo().getNum();
//            showToast(parentRemarks+" 的 "+remarks+" 回路号："+num+" 返回"+" 状态为："+curState);
            HDLLog.Log(parentRemarks + " 的 " + remarks + " 回路号：" + num + " 返回" + " 状态为：" + curState);
            if (event.getCurtainCtrlBackInfo().getAppliancesInfo().getDeviceType() == HDLApConfig.TYPE_CURTAIN_MODULE) {
                showMessage = "";
                //判断是否为窗帘模块
                switch (curState) {
                    case CurtainCtrlParser.curtainOff:
                        showMessage = "窗帘关";
                        curtainBtn.setText(showMessage);
                        curText1.setText(showMessage);
                        curtainState = curState;
                        HDLLog.Log("窗帘控制 ：窗帘关" + "  回路号：" + num);
                        break;
                    case CurtainCtrlParser.curtainOn:
                        showMessage = "窗帘开";
                        curtainBtn.setText(showMessage);
                        curText1.setText(showMessage);
                        curtainState = curState;
                        HDLLog.Log("窗帘控制 ：窗帘开" + "  回路号：" + num);
                        break;
                    case CurtainCtrlParser.curtainPause:
                        showMessage = "窗帘暂停";
                        curtainBtn.setText(showMessage);
                        curText1.setText(showMessage);
                        curtainState = curState;
                        HDLLog.Log("窗帘控制 ：窗帘暂停" + "  回路号：" + num);
                        break;
                }
                showToast(showMessage);
            } else {
                //开合帘或卷帘 显示百分比
                curtainBtn5.setText("窗帘开到" + curState + "%");
                curText2.setText("窗帘开到" + curState + "%");
                curtainState = curState;
            }

        }

    }
```