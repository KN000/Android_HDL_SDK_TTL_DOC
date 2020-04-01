# 生成设备列表


### 添加虚拟回路数据


 ###  SDK目前支持的大类：小类
 * Configuration.LIGTH_BIG_TYPE
 * Configuration.CURTAIN_BIG_TYPE
 * Configuration.AIR_BIG_TYPE
 * Configuration.AUDIO_BIG_TYPE
 * Configuration.LOGIC_BIG_TYPE
 * Configuration.GLOBAL_LOGIC_BIG_TYPE
 * Configuration.SECURITY_BIG_TYPE
 * Configuration.COMMON_SWITCH_BIG_TYPE
 *
 * 灯光类1：0 ，1，9，10
 * 窗帘类2：0，1，2
 * 传感器5：0~24
 * 空调类7：0，3
 * 背景音乐功能9：0
 * 逻辑功能12：0
 * 全局场景17：0
 * 安防10：0
 * 通用开关100：0
 *
 * 该方法应用于提供项目交付前的提取批量数据生成好数据。
 * 模拟生成设备回路数据，在项目不支持简易编程搜索情况下，可以通过该方法，先快捷生成目标数据 得到 List<DevicesData> 格式的设备列表数据
 *
 * 上层做本地保存或者云端备份，App启动时读取恢复
 * 每次启动时先加载生成好的设备列表数据，然后在 SDK 初始化后，赋值给 HDLDeviceManager.devicesDataList
 

### 一、添加不同类型设备回路
**代码范例**

```java
        /**
         * 添加设备回路
         * 如果存在相同子网号 设备号，则当成混合模块添加到该回路下，不存在则新添加模块
         * @param bigType 回路大类
         * @param littleType 回路小类
         * @param mSubnetID  回路子网ID
         * @param mDeviceID  回路设备ID
         * @param mChannelNum  回路号
         * @param mChannelRemark 回路备注 例如：窗帘、吊灯、厕所灯、电视
         * @param parentRemarks  当前回路模块的备注 例如: 酒店RCU模块
         * @return
         */
        //如果存在相同子网号 设备号，则当成混合模块添加到该回路下，不存在则新添加模块
        DevicesData mDevicesData = DeviceParser.addDevicesListWithoutSearching(bigType, littleType, mSubnetID, mDeviceID, mChannelNum, remarksString, parentRemarks);
        if (mDevicesData != null) {
            showToast("添加成功");
        } else {
            showToast("添加失败");
        }


```

添加成功后同样会收到DevicesInfoEvent订阅事件，可以通过event.getDesDataList() 来获取生成好的数据。或者直接通过HDLDeviceManager.devicesDataList获取

**代码范例**

```java
    /**
     * ====================EventBus 订阅事件 ====================
     */
    @Subscribe(threadMode = ThreadMode.MAIN)
    public void onDevicesInfoEventMain(DevicesInfoEvent event) {
        mAllDevicesList.clear();
        mProgressDialog.dismiss();
        if (!event.isSuccess()) {
            Toast.makeText(MainActivity.this, "搜索超时，请重新再试", Toast.LENGTH_SHORT).show();
            tvResult.setText("搜索超时，请重新再试");
            return;
        }
        mDevicesDataList = event.getDesDataList();
        updateDeviceListView();
    }

```

### 二、添加场景
**代码范例**

```java
 
    /**
     * 添加场景设备回路 需指定 场景区号 和 场景号
     * 如果存在相同子网号 设备号，则当成混合模块添加到该回路下，不存在则新添加模块
     *
     * @param mSubnetID
     * @param mDeviceID
     * @param mAreaNum       //场景 区域号
     * @param mAreaSceneNum  //场景 当前区域场景号
     * @param mChannelRemark 读取场景的备注名称 例如: 入住、起床模式、阅读模式
     * @param parentRemarks  当前回路模块的备注 例如: 酒店RCU模块
     * @return
     */
    private void AddScenesDevices(int mSubnetID, int mDeviceID, int mAreaNum, int mAreaSceneNum, String mChannelRemark, String parentRemarks) {
        //添加场景
        DevicesData mScenesData = DeviceParser.addScenesDevicesListWithoutSearching(mSubnetID, mDeviceID, mAreaNum, mAreaSceneNum, mChannelRemark, parentRemarks);
    }


```

添加成功后同样会收到DevicesInfoEvent订阅事件

### 三、初始化设备列表

 * 该方法应用于提供项目交付前的提取批量数据生成好数据。
 * 模拟生成设备回路数据，在项目不支持简易编程搜索情况下，先快捷生成目标数据 得到 List<DevicesData> 格式的设备列表数据

**代码范例**

```java
   /**
     * 初始化 HDLSDK
     */
    private void initHDLSDK() {
        /**SDK初始化 配置串口 路径和波特率参数*/
        //HDL_PATH_NAME (串口路径)
        //HDL_BAUDRATE  (波特率)，波特率默认115200
        HDLTtlSdk.init(this, HDL_PATH_NAME, HDL_BAUDRATE);
        //HDLTtlSdk.setHDLLogOpen(false);//配置是否开启SDK打印日志，默认为打开


        /*****不支持简易编程搜索的项目，上层保存读取加载 赋值给SDK*************************/
        List<DevicesData> mDevicesDataList = new ArrayList<>();
        .......
        //mDevicesDataList 上层做本地保存或者云端备份，App启动时读取恢复 加载该设备列表
        .......
        HDLDeviceManager.devicesDataList = mDevicesDataList;
        /*******************不支持简易编程搜索的项目***********************************/
    }
```