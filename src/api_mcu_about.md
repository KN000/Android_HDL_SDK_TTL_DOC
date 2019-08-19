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

