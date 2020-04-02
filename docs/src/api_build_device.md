# 非搜索设备解决方案
## 生成设备列表 手动添加设备

手动添加，模拟生成设备列表（在不支持简易编程搜索情况下使用，例如：RCU酒店模块）


### 说明

```java
   /** SDK目前支持的大类：小类
 * Configuration.LIGTH_BIG_TYPE
 * Configuration.CURTAIN_BIG_TYPE
 * Configuration.AIR_BIG_TYPE
 * Configuration.AUDIO_BIG_TYPE
 * Configuration.LOGIC_BIG_TYPE
 * Configuration.GLOBAL_LOGIC_BIG_TYPE
 * Configuration.SECURITY_BIG_TYPE
 * Configuration.COMMON_SWITCH_BIG_TYPE
 * 
 * 灯光类1：0 ，1，9，10 （后面为小类ID号）
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
 */

```

