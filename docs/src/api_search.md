# 搜索设备

### 一、搜索设备

**接口描述**

HDLCommand.getHomeDevices()

调用该接口，会清空原本地设备列表数据，重新搜索设备（可以搜索出支持简易编程搜索的设备）。

**代码范例**

发送搜索设备请求
```java
    /**全部重新搜索,清空原设备列表数据*/
    HDLCommand.getHomeDevices();
```

搜索设备回调事件

搜索超时或搜索成功，都会收到DevicesInfoEvent订阅事件。

搜索成功后，会返回搜到到的所有模块设备数据，通过event.getDesDataList()取出，后面的所有的控制设备操作都需要用到这个List里面的AppliancesInfo设备参数，SDK做了本地保存操作，同样开发者可以根据需要自己另外保存。

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

### 二、手动添加安防模块设备

**接口描述**

HDLDeviceManager.addSecurityDevicesManually(int mSubnetID, int mDeviceID, int mChannelNum, String mRemarks)

调用该接口，可以手动添加安防模块（不支持简易编程搜索的设备，只能手动添加）。

需要传入该安防模块当前的子网号，设备号，回路号，以及它的备注名。

**代码范例**


```java
    .......
    //isSuccess 返回true则代表添加成功
    isSuccess = HDLDeviceManager.addSecurityDevicesManually(subnetID, deviceID, channelNum, "安防模块1");
    .......

```

添加成功后同样会收到DevicesInfoEvent订阅事件，更新返回当前所有的设备列表数据

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


### 三、手动添加音乐模块设备

**接口描述**

HDLDeviceManager.addAudioDevicesManually(int mSubnetID, int mDeviceID, int mChannelNum, String mRemarks)

调用该接口，可以手动添加安防模块（不支持简易编程搜索的设备，只能手动添加）。

需要传入该安防模块当前的子网号，设备号，回路号，以及它的备注名。

**代码范例**


```java
    .......
    //isSuccess 返回true则代表添加成功
    isSuccess = HDLDeviceManager.addAudioDevicesManually(subnetID, deviceID, channelNum, "音乐模块1");
    .......

```

添加成功后同样会收到DevicesInfoEvent订阅事件，更新返回当前所有的设备列表数据

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

### 四、加载本地设备

**接口描述**

HDLDeviceManager.getLocalDevicesDataList()

调用该接口，会读取加载SDK本地保存的设备列表数据。

**代码范例**


```java
    /**
     * 读取和加载本地数据
     */
    private void getLocalDevicesDataList() {
        mDevicesDataList = HDLDeviceManager.getLocalDevicesDataList();//加载本地数据
        if (mDevicesDataList.size() > 0) {
            updateDeviceListView();
        } else {
            showToast("本地数据为空");
        }
    }

```