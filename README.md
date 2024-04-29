### 编译安装

1. #### 编译
   FreeSWITCH和NlsSdkCpp3.x路径根据自己情况修改

```
make
```

2. ####安装

#把模块复制到freeswitch的mod目录下
```bash
cp mod_aliasr.so /usr/local/freeswitch/mod
```
#把NlsSdk3.X/lib下的so复制到/usr/local/lib/
```bash
cp NlsSdk3.X/lib/libnls_sdk_cpp.so /usr/local/lib
```
#编辑modules.conf.xml添加mod_aliasr模块
vim /usr/local/freeswitch/conf/autoload_configs/modules.conf.xml
<load module="mod_aliasr"/>
#把aliasr.con.xml参数配置好后复制到/usr/local/freeswitch/conf/autoload_configs/
```bash
cp aliasr.conf.xml /usr/local/freeswitch/conf/autoload_configs/
```

3. #### 验证
   启动freeswitch查看mod_aliasr是否加载成功

```
freeswitch -nc -nonat
fs_cli -x "show modules"|grep asr
application,start_aliasr,mod_aliasr,/usr/local/freeswitch/mod/mod_aliasr.so
application,stop_aliasr,mod_aliasr,/usr/local/freeswitch/mod/mod_aliasr.so
```

### 使用

1. 申请阿里云AccessKey和Secret
2. fs_cli执行

start_aliasr参数:

```
originate user/1001 'start_aliasr,echo' inline
```

3. dialplan执行

```
<extension name="aliasr">
    <condition field="destination_number" expression="^.*$">
        <action application="answer"/>
        <action application="start_aliasr" data=""/>
        <action application="echo"/>
    </condition>
</extension>
```

#### 开发

订阅`CUSTOM aliasr_start` `CUSTOM aliasr_update` `CUSTOM aliasr_stop` 事件
fs_cli可以通过`/event plain Custom aliasr_start  aliasr_update aliasr_stop`订阅事件
识别结果通过esl输出

```
RECV EVENT
Event-Subclass: start_aliasr
Event-Name: CUSTOM
Core-UUID: dbc6fb6a-16e6-44cb-8be8-a49397cc3c5f
FreeSWITCH-Hostname: telegant
FreeSWITCH-Switchname: telegant
FreeSWITCH-IPv4: 10.10.16.180
FreeSWITCH-IPv6: ::1
Event-Date-Local: 2021-01-25 16:33:20
Event-Date-GMT: Mon, 25 Jan 2021 08:33:20 GMT
Event-Date-Timestamp: 1611563600014063
Event-Calling-File: mod_aliasr.cpp
Event-Calling-Function: onAsrSentenceEnd
Event-Calling-Line-Number: 215
Event-Sequence: 2485
Channel-Call-UUID: 8a53b863-e6fc-46e1-902b-7b1931c47164
ASR-Response: {"header":{"namespace":"SpeechTranscriber","name":"SentenceEnd","status":20000000,"message_id":"0ca84cbeed884ca39c88c0c5ae4edbb4","task_id":"97f77f8f53f14eef8be5469375051d81","status_text":"Gateway:SUCCESS:Success."},"payload":{"index":3,"time":4950,"result":"你好。","confidence":0.38665345311164856,"words":[],"status":20000000,"gender":"","begin_time":3600,"stash_result":{"sentenceId":0,"beginTime":0,"text":"","currentTime":0},"audio_extra_info":"","sentence_id":"92887f8e4444437598baeb3768e87035","gender_score":0.0}}
Channel: sofia/internal/8000@cc.com


RECV EVENT
Event-Subclass: stop_aliasr
Event-Name: CUSTOM
Core-UUID: dbc6fb6a-16e6-44cb-8be8-a49397cc3c5f
FreeSWITCH-Hostname: telegant
FreeSWITCH-Switchname: telegant
FreeSWITCH-IPv4: 10.10.16.180
FreeSWITCH-IPv6: ::1
Event-Date-Local: 2021-01-25 16:34:13
Event-Date-GMT: Mon, 25 Jan 2021 08:34:13 GMT
Event-Date-Timestamp: 1611563653232813
Event-Calling-File: mod_aliasr.cpp
Event-Calling-Function: onAsrChannelClosed
Event-Calling-Line-Number: 331
Event-Sequence: 2495

RECV EVENT
Event-Subclass: update_aliasr
Event-Name: CUSTOM
Core-UUID: dbc6fb6a-16e6-44cb-8be8-a49397cc3c5f
FreeSWITCH-Hostname: telegant
FreeSWITCH-Switchname: telegant
FreeSWITCH-IPv4: 10.10.16.180
FreeSWITCH-IPv6: ::1
Event-Date-Local: 2021-01-25 16:34:50
Event-Date-GMT: Mon, 25 Jan 2021 08:34:50 GMT
Event-Date-Timestamp: 1611563690152634
Event-Calling-File: mod_aliasr.cpp
Event-Calling-Function: onAsrTranscriptionResultChanged
Event-Calling-Line-Number: 249
Event-Sequence: 2512
Channel-Call-UUID: 8a53b863-e6fc-46e1-902b-7b1931c47164
ASR-Response: {"header":{"namespace":"SpeechTranscriber","name":"TranscriptionResultChanged","status":20000000,"message_id":"06bf1659fc904abcb95c727e7fb143a2","task_id":"3d7563f486a74aa28b3d50256eae0958","status_text":"Gateway:SUCCESS:Success."},"payload":{"index":1,"time":6340,"result":"结果是100还是150 ？","confidence":0.45189642906188965,"words":[],"status":20000000}}
Channel: sofia/internal/8000@cc.com

```

ASR-Response: asr识别返回结果 Channel: 当前Channel Name 

