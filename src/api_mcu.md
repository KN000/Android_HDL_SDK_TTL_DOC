# MCU通信


### MCU控制统一回调

MCU的所有请求回调，超时、出错或成功都会统一收到MCUFeedBackEvent订阅事件，用户通过获取返回操作码（event.getMCUDataBean().command）来判断是哪种回调信息，从而实现处理不同回调的数据。


**代码范例**

```java
    /**
     * MCU控制回调Event
     *
     * @param event
     */
    @Subscribe(threadMode = ThreadMode.MAIN)
    public void onMCUFeedBackEventMain(MCUFeedBackEvent event) {
        if (event.getEventCode() == EventCode.FAILURE_TIMEOUT) {
            if (TextUtils.isEmpty(event.getError())) {
                showToast("MCU请求超时，请稍后再试");
            } else {
                showToast(event.getError());
            }
            return;
        }

        switch (event.getMCUDataBean().command) {
            case MCUConstants.MCU_COMMAND_SEND_BACK:    //
                //透传返回的数据
                handleMCUPassThroughDataBack(event);
                break;
            case MCUConstants.MCU_READ_CONFIGURATION_BACK:// 读配置返回
            case MCUConstants.MCU_WRITE_CONFIGURATION_BACK:// 写配置信息返回
                handleMCUConfigurationBack(event);
                break;
            // case MCUConstants.MCU_DETECT_PACKET_LOSS_STATUS_BACK:// 检测丢包状态返回
            //     //待处理
            //     handleDetectPacketLossStatusBack(event);
                break;
            case MCUConstants.MCU_REQUEST_UPGRADE_BACK:   // 请求升级返回
                handleRequestUpgradeBack(event);
                break;
            case MCUConstants.MCU_RESTART_BACK:     //0x86 重启返回
                handleRestartBack(event);
                break;
            default:
                break;
        }
    }

```

### 一、读配置参数

**HDLCommand.mcuReadConfiguration()**

**接口描述**

调用该接口，可以读取MCU的配置参数，协议类型、波特率、数据位、校验位和停止位 。

**代码范例**

发送读取MCU配置请求
```java
    //读取MCU配置请求
     HDLCommand.mcuReadConfiguration();
```

读配置信息回调

当监听到command为MCUConstants.MCU_READ_CONFIGURATION_BACK时，为读取配置信息的回调，成功获取MCU配置数据处理并显示；

MCU的配置参数：协议类型、波特率、数据位、校验位和停止位 ，都可以从返回event中获取。

获取方法：event.getMCUDataBean().getMCUConfigurationBean()

**代码范例**

```java
    .......
    case MCUConstants.MCU_READ_CONFIGURATION_BACK:// 写配置信息返回
        handleMCUConfigurationBack(event);//处理MCU读写配置返回数据
    .......

    /**
     * 处理MCU读写配置返回数据
     *
     * @param event
     */
    private void handleMCUConfigurationBack(MCUFeedBackEvent event) {
        if (event.getEventCode() != EventCode.SUCCESS) {
            tv_config_mesSetText("MCU读写配置出错，返回数据异常");
            return;
        }
        tv_config_mesSetText("成功获取配置");
        bGetMCUConfiguration = true;
        updateMCUConfiguration(event.getMCUDataBean().getMCUConfigurationBean());//获取配置参数并显示
    }
```



### 二、写配置参数

**HDLCommand.mcuWriteConfiguration(byte[] sendBytes)**

**接口描述**

调用该接口，可以修改MCU的配置参数，协议类型、波特率、数据位、校验位和停止位 。

**代码范例**

发送写MCU配置请求，开发者通过修改更新mMCUConfigurationBean对应的参数后，直接调用接口并参入参数即可。


```java
    .......
    private MCUConfigurationBean mMCUConfigurationBean;//配置参数
    .......

    .......
    mMCUConfigurationBean.setMcuDataBit(0);//修改数据位参数
    .......

    //MCU写配置请求
    HDLCommand.mcuWriteConfiguration(mMCUConfigurationBean.getMCUConfigurationSendBytes());
```

写配置信息回调

当监听到command为MCUConstants.MCU_READ_CONFIGURATION_BACK时，为读取配置信息的回调，成功获取MCU配置数据处理并显示；

MCU的配置参数：协议类型、波特率、数据位、校验位和停止位 ，都可以从返回event中获取。

获取方法：event.getMCUDataBean().getMCUConfigurationBean()

**代码范例**

```java
    .......
    case MCUConstants.MCU_WRITE_CONFIGURATION_BACK:// 写配置信息返回
        handleMCUConfigurationBack(event);//处理MCU读写配置返回数据
    .......

    /**
     * 处理MCU读写配置返回数据
     *
     * @param event
     */
    private void handleMCUConfigurationBack(MCUFeedBackEvent event) {
        if (event.getEventCode() != EventCode.SUCCESS) {
            tv_config_mesSetText("MCU读写配置出错，返回数据异常");
            return;
        }
        tv_config_mesSetText("成功获取配置");
        bGetMCUConfiguration = true;
        updateMCUConfiguration(event.getMCUDataBean().getMCUConfigurationBean());//获取配置参数并显示
    }
```


### 三、MCU重启

**HDLCommand.mcuSendRestart()**

**接口描述**

调用该接口，可以让MCU重启。

**代码范例**

发送MCU重启请求
```java
    //MCU重启请求
    HDLCommand.mcuSendRestart();
```

MCU重启回调

当监听到command为MCUConstants.MCU_RESTART_BACK时，为MCU重启回调；

**代码范例**

```java
    .......
    case MCUConstants.MCU_RESTART_BACK:     //重启返回
                handleRestartBack(event);
    .......

    /**
     * 处理MCU请求重启返回数据
     *
     * @param event
     */
    private void handleRestartBack(MCUFeedBackEvent event) {
        if (event.getEventCode() != EventCode.SUCCESS) {
            showToast("MCU请求重启返回错误，重启失败");
            return;
        }
        if (bUpdating) {
            bUpdating = false;
            tv_update_mes.setText("MCU重启成功,升级完成！");
            tv_other_mesSetText("MCU重启成功,升级完成！");
        } else {
            tv_other_mesSetText("MCU重启成功！");
        }
    }
```




### 四、升级MCU固件

**HDLCommand.mcuRequestUpgradeWithFile(byte[] upgradeFileDatas)**

**接口描述**

调用该接口，可以升级MCU固件。

**代码范例**

开发者首先需选择本地bin升级文件，并转换成byte[]类型升级文件数据，然后调用升级接口，并传入升级文件参数即可；

```java
    .......
    private byte[] upgradeFileDatas;//升级文件数据
    .......

    .......
    //读取升级文件
    fis = new FileInputStream(file);
    int length = fis.available();
    upgradeFileDatas = new byte[length];
    fis.read(upgradeFileDatas);
    fis.close();
    .......

    .......
    //开始升级
    if (upgradeFileDatas == null) {
        showToast("请先选择升级文件");
        return;
    }
    HDLCommand.mcuRequestUpgradeWithFile(upgradeFileDatas);//传入升级文件并开始升级
    .......
```

**升级进度监听**

首先activity继承IMcuOtaListener，并实现对应的接口方法；

**代码范例**
```java
    .......
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        .......
        HDLSerialPortCore.setIMcuOtaListener(this);//设置监听
    }
    .......

    .......
    @Override
    public void onSuccess() {
        sendRestartTimer();//升级10S超时提示
        HDLLog.Log("IMcuOtaListener onSuccess");
        runOnUiThread(new Runnable() {  //要在主线程更新UI
            public void run() {
                tv_update_mesSetText("升级成功，等待重启...");
            }
        });
    }

    @Override
    public void onFailure(int code, final String error) {
        HDLLog.Log("IMcuOtaListener onFailure");
        runOnUiThread(new Runnable() {
            public void run() {
                tv_update_mesSetText("升级失败： " + error);
            }
        });

    }

    @Override
    public void onProgress(final int progress) {
        HDLLog.Log("IMcuOtaListener onProgress：" + progress);
        runOnUiThread(new Runnable() {
            public void run() {
                tv_update_mes.setText("升级进度：" + progress + "%");
            }
        });

    }
    .......
```

升级成功后，MCU会自动重启,监听到重启完毕，才代表完成升级；


**MCU升级状态回调**

当监听到command为MCUConstants.MCU_REQUEST_UPGRADE_BACK时，为MCU升级状态回调；


**代码范例**

```java
    .......
    case MCUConstants.MCU_REQUEST_UPGRADE_BACK:   // 请求升级返回
                handleRequestUpgradeBack(event);
    .......

     /**
     * 处理MCU请求升级返回数据
     *
     * @param event
     */
    private void handleRequestUpgradeBack(MCUFeedBackEvent event) {
        if (event.getEventCode() != EventCode.SUCCESS) {
            tv_update_mesSetText("MCU请求升级出错");
            return;
        }
        bUpdating = true;
        tv_update_mesSetText("MCU请求升级成功，开始升级...");
    }

```


### 五、发送透传数据

**HDLCommand.mcuSendTransparentData(byte[] sendBytes)**

**接口描述**

调用该接口，可以发送透传数据，不走bus协议，直接转发，不做数据格式处理。

调用该接口之后收到的数据，默认走数据透传不做格式处理，不需要时可调接口关闭。

**代码范例**

```java
    .......
    // 发送透传数据
    byte[] sendBytes = new byte[]{(byte) 0x00, (byte) 0x00, (byte) 0x00, (byte) 0x00};
    HDLCommand.mcuSendTransparentData(sendBytes);
    .......
```

透传数据接收

当监听到command为MCUConstants.MCU_COMMAND_SEND_BACK时，为返回接收到的透传数据；

接收的透传数据可以直接从event获取，event.getMCUDataBean().receiveBytes。

**代码范例**

```java
    .......
    case MCUConstants.MCU_COMMAND_SEND_BACK:    //
        //透传返回的数据
        handleMCUPassThroughDataBack(event);
    .......

    /**
     * 处理MCU透传回调数据
     * 透传数据，不走bus协议，直接转发，不做数据格式处理
     *
     * @param event
     */
    private void handleMCUPassThroughDataBack(MCUFeedBackEvent event) {
        if (event.getEventCode() != EventCode.SUCCESS) {
            tv_other_mesSetText("MCU透传数据出错，返回数据异常");
            return;
        }
        String receiveString = HDLStringUtils.ByteArrToHex(event.getMCUDataBean().receiveBytes, 0, event.getMCUDataBean().receiveBytes.length);

        tv_other_mes.setText("收到的透传数据：" + receiveString);
        HDLLog.Log("收到的透传数据：" + receiveString);
    }
```

### 六、开启和关闭透传数据

**HDLCommand.setHDLPassThroughOpen(boolean bOpen)**

**接口描述**

调用该接口，可以开启和关闭透传数据，关闭数据透传默认走bus协议；其他协议下，开发者可以打开数据透传，自己自定义收发数据。

注：SDK每次重新初始化启动后，默认都是关闭，如果要数据透传，需要初始化完毕后调用改接口打开。

**代码范例**

```java
    .......
    //关闭透传数据
     HDLCommand.setHDLPassThroughOpen(false);
    .......
```