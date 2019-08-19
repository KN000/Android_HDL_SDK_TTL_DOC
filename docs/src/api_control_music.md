# 音乐类模块

**音乐类模块支持搜索和控制的类型**

背景音乐模块（0） 


### 一、背景音乐控制

**HDLCommand.audioCtrl(AppliancesInfo info, int type)**

**接口描述**

info：需要控制设备的参数

type：需要控制功能命令参数

调用该接口，可以控制背景音乐模块，获取当前歌曲信息、播放/暂停、播放/停止、上一首、下一首、切换播放模式、获取下一列表、获取上一列表。

**代码范例**

```java
    .......
    HDLCommand.audioCtrl(appliancesInfo, HDLAudio.GET_AUDIO_CURRRENT_INFO);//获取当前音乐信息。返回当前歌曲、所有信息。
    HDLCommand.audioCtrl(appliancesInfo, HDLAudio.GET_AUDIO_MODE);//获取当前音乐播放模式。仅返回单曲播放等播放模式。
    .......
    .......
    //播放或暂停
    HDLCommand.audioCtrl(appliancesInfo, HDLAudio.SET_AUDIO_PLAYPAUSE);
    .......
    .......
    //播放或停止
    HDLCommand.audioCtrl(appliancesInfo, HDLAudio.SET_AUDIO_PLAYSTOP);
    .......
    .......
    //上一首
    HDLCommand.audioCtrl(appliancesInfo, HDLAudio.SET_PRE_SONG);
    .......
    .......
    //下一首
    HDLCommand.audioCtrl(appliancesInfo, HDLAudio.SET_NEXT_SONG);
    .......
    .......
    //切换播放模式
    HDLCommand.audioCtrl(appliancesInfo, HDLAudio.SET_AUDIO_MODE_UP);//播放模式+
    //HDLCommand.HDLaudioCtrl(CtrlAudioActivity.this,appliancesInfo,HDLAudio.SET_AUDIO_MODE_DOWN);//播放模式-
    .......
    .......
    //获取下一列表，当前音乐会停止播放
    HDLCommand.audioCtrl(appliancesInfo, HDLAudio.SET_NEXT_LIST);
    .......
    .......
    //获取上一列表，当前音乐会停止播放
    HDLCommand.audioCtrl(appliancesInfo, HDLAudio.SET_PRE_LIST);
    .......

```


**HDLCommand.audioCtrl(AppliancesInfo info, int type, int value)**

**接口描述**

调用该接口，可以控制背景音乐模块，获取当前播放列表、控制设备音量。

**代码范例**

```java

    .......
    //获取当前播放列表，此方法如果在歌曲播放状态时调用则会导致歌曲停止播放，硬件设计如此
    HDLCommand.audioCtrl(appliancesInfo, HDLAudio.GET_AUDIO_LIST, curListNum);
    .......               
    .......
    //设置播放音量
    HDLCommand.audioCtrl(appliancesInfo, HDLAudio.SET_AUDIO_VOL, 0);//音量最小：0。小于0，SDK不处理
    .......
```



**HDLCommand.audioCtrl(AppliancesInfo info, int type, int listId, int songId)**

**接口描述**

listId：当前列表号

songId：要播放的列表中歌曲对应的歌曲ID

调用该接口，可以控制背景音乐模块，选择指定播放的歌曲。

**代码范例**

```java      
    .......
    //设置播放音量
    HDLCommand.audioCtrl(appliancesInfo, HDLAudio.SET_CHOOSE_PLAY_SONG, curListNum, position);
    .......
```



**查询状态回调事件监听**

背景音乐模块查询、控制超时或成功，都会收到AudioInfoEvent订阅事件；

开发者可以在所需要获取状态回调的地方，订阅该事件；
并通过比较前设备和回调状态设备的子网号（getDeviceSubnetID）、设备号（getDeviceDeviceID）是否一致，从而判断该消息事件为当前设备查询状态返回的消息，进行下一步处理，不一致则过滤不处理。

根据返回的event.getType()类型，判断是什么类型状态返回。

例：getType()等于HDLAudio.CALLBACK_CURRENT_VOLUME 的话，为返回音量类型参数参数

输出打印：HDLLog.Log("当前音量值：" + event.getAudioInfoInt());


**代码范例**
```java
    /**
     * 音乐模块状态回调Event
     *
     * @param event
     */
    @Subscribe(threadMode = ThreadMode.MAIN)
    public void onAudioEventMain(AudioInfoEvent event) {
        //判断是否为本音乐模块的子网号，设备号
        if (event.getAppliancesInfo().getDeviceSubnetID() == appliancesInfo.getDeviceSubnetID()
                && event.getAppliancesInfo().getDeviceDeviceID() == appliancesInfo.getDeviceDeviceID()
        ) {
            HDLLog.Log("onAudioEventMain: " + event.getType());
            switch (event.getType()) {
                case HDLAudio.CALLBACK_SONG_NAME_LIST:
                    listString.clear();
                    for (int i = 0; i < event.getSongNameList().size(); i++) {
                        listString.add(event.getSongNameList().get(i));
                    }
                    adapter.notifyDataSetChanged();
                    break;
                case HDLAudio.CALLBACK_CURRENT_VOLUME:
                    HDLLog.Log("当前音量值：" + event.getAudioInfoInt());
                    break;
                case HDLAudio.CALLBACK_AUDIO_LIST_NUM:
                    int[] listNum = event.getAudioListInfo();
                    curListNum = listNum[0];
                    HDLLog.Log("当前列表号：" + listNum[0] + " 当前共有列表数：" + listNum[1]);
                    if (isInit) {
                        isInit = false;//此操作为仅初始化才请求获取当前音乐列表，厂商可以自行决定何时获取音乐列表
                        HDLCommand.audioCtrl(appliancesInfo, HDLAudio.GET_AUDIO_LIST, curListNum);//获取当前播放列表，此方法如果在歌曲播放状态时调用则会导致歌曲停止播放，硬件设计如此
                    }
                    break;
                case HDLAudio.CALLBACK_CURRENT_LIST_NAME:
                    HDLLog.Log("当前列表名：" + event.getAudioInfoStr());
                    break;
                case HDLAudio.CALLBACK_CURRENT_SONG_NUM:
                    int[] songNum = event.getAudioListInfo();
                    HDLLog.Log("当前歌曲号：" + songNum[0] + " 当前共有歌曲数：" + songNum[1]);
                    break;
                case HDLAudio.CALLBACK_CURRENT_SONG_NAME:
                    HDLLog.Log("当前歌曲名：" + event.getAudioInfoStr());
                    curSongNameTv.setText("当前歌曲名：" + event.getAudioInfoStr());
                    break;
                case HDLAudio.CALLBACK_CURRENT_SONG_INFO:
                    int[] songInfo = event.getAudioListInfo();
                    //songInfo[0],songInfo[1]获得的值为秒，如songInfo[0]=250，即歌曲总时长为250秒。songInfo[2]获得的值为：1、2、3。1：停止，2：播放，3：暂停。
                    String curStatus;
                    switch (songInfo[2]) {
                        case 1:
                            curStatus = "停止";
                            break;
                        case 2:
                            curStatus = "播放";
                            break;
                        case 3:
                            curStatus = "暂停";
                            break;
                        default:
                            curStatus = "未知";
                            break;
                    }
                    HDLLog.Log("当前歌曲总时长：" + songInfo[0] + "秒 ，当前歌曲已播放时长：" + songInfo[1] + "秒， 当前歌曲状态：" + curStatus);
                    curSongInfoTv.setText("当前歌曲总时长：" + songInfo[0] + "秒 ，当前歌曲已播放时长：" + songInfo[1] + "秒， 当前歌曲状态：" + curStatus);
                    break;
                case HDLAudio.CALLBACK_CURRENT_MODE:
                    String curMode;
                    switch (event.getAudioInfoInt()) {
                        case 1:
                            curMode = "单曲播放";
                            break;
                        case 2:
                            curMode = "单曲循环";
                            break;
                        case 3:
                            curMode = "连续播放";
                            break;
                        case 4:
                            curMode = "连播循环";
                            break;
                        default:
                            curMode = "未知";
                            break;

                    }
                    modeBtn.setText(curMode);
                    break;
                default:
                    break;
            }
        }


    }

```

**注：HDLAudio.CALLBACK_CURRENT_SONG_INFO**

```java
HDLLog.Log("当前歌曲总时长：" + songInfo[0] + "秒 ，当前歌曲已播放时长：" + songInfo[1] + "秒， 当前歌曲状态：" + curStatus);
```

这里虽然可以获取，当前歌曲已播放时长，但APP上音乐播放进度条，不能根据这里的播放时长显示播放进度，因为有的音乐面板不会主动发送当前播放时长，所以开发者应自己创建一个定时器，定时更新播放时长时间值和进度条。

