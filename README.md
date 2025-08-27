# FreeSwitch Mod_FunAsr 模块

**很多人都对FreeSWITCH和ASR对接比较感谢兴趣，这里提供FunAsr mod模块直接对接阿里开源的FunAsr语音Asr识别模型，大家可以基于此模块实现检测早期媒体音,实现手机号空号识别+各种前期手机号异常状态功能**

mod_fun_asr.so是模块主程序，该模块需要授权，授权价格3K，模块一次性永久授权，包售后,可开发票。如有其他定制化模块需求，请加微信或进入https://www.callwg.com/ 了解更多详情。


**如果您需要神经网络算法实现检测振铃音的无需ASR大模型，可以访问https://github.com/smswg/FreeSwitch-Mod_CheckPhone 来这个仓库看一下,或加客服了解。**

![](https://www.callwg.com/templets/callgw/images/ma.png)

## `mod_fun_asr模块介绍`

此模块由合肥标通科技有限公司开发，基于FreeSwitch1.10.9版本开发,可支持最新版本的FreeSwitch，使用C++11原生编写。本模块基于阿里达摩院开源的FunAsr对接实现实时Asr识别需求，经大量生产高并发测试非常稳定，特别适合需要检测早期媒体音判断是否是空号等场景,如果您是做外呼系统的相关科技公司可以直接采购本模块,节省大量的踩坑成本。本公司有成熟的callwg语音呼叫系统，欢迎各位老板前来采购。


## `mod_fun_asr模块使用教程`
1.首先下载mod_fun_asr.so文件到FreeSwitch运行目录,正常目录是/usr/local/freeswitch/mod/。
2.复制funasr.xml文件到/usr/local/freeswitch/conf/autoload_configs/目录下，并修改相关参数。

```
<configuration name="funasr.conf" description="mod_fun_asr configuration">
    <settings>
	    <!-- 注意ws不支持wss地址,为了效率考虑直接使用ws -->
        <param name="wsUrl" value=""/>
		<!-- 热词 :后面是权重 注意&quot;是转义"字符串 -->
		<param name="hotWords" value="{&quot;阿里巴巴&quot;:20,&quot;通义实验室&quot;:30}"/>
		<!-- 推送asr识别结果到接口 -->
        <param name="pushUrl" value=""/>
    </settings>
</configuration>
```

3.修改/usr/local/freeswitch/conf/autoload_configs/modules.conf.xml 增加一行<load module="mod_fun_asr"/>,重启fs系统生效模块。

4.模块使用命令如下:

```
originate {origination_caller_id_name=955555,origination_caller_id_number=955555,absolute_codec_string=^^:PCMU:PCMA,leg_timeout=30,ignore_early_media=false}user/4000 'start_fun_asr,playback:/opt/record/2023-03-29/start.wav' inline
```

命令解释：使用955555号码呼叫分机4000并同时开启start_fun_asr模块,等4000分机接通后播放start.wav音乐。

## `mod_fun_asr模块事件结果`

1.上面的命令执行成功以后，会不断地生成Asr识别的结果，分别通过Http Post json到业务接口或Esl事件方式通知,可以二选一方式。

Esl事件如下：

​		update_asr中间识别结果事件

```
Event-Subclass: update_asr
Event-Name: CUSTOM
Unique-ID: 58a08a69-7858-407a-be69-679150d34193
FreeSWITCH-Hostname: MiWiFi-R3D-srv
FreeSWITCH-Switchname: MiWiFi-R3D-srv
FreeSWITCH-IPv4: 192.168.31.164
FreeSWITCH-IPv6: ::1
Event-Date-Local: 2017-12-10 11:30:32
Event-Date-GMT: Sun, 10 Dec 2017 03:30:32 GMT
Event-Date-Timestamp: 1512876632835590
Event-Calling-File: mod_fun_asr.cpp
Event-Calling-Function: OnResultDataRecved
Event-Calling-Line-Number: 55
Event-Sequence: 914
ASR-Response: {"is_final":false,"mode":"2pass-online","text":"的都","wav_name":"asr"}
Channel: sofia/external/linphone@192.168.31.210
```

​		stop_asr识别结束事件 建议使用这个事件作为最终识别结果

​		

```
Event-Subclass: stop_asr
Event-Name: CUSTOM
Unique-ID: 58a08a69-7858-407a-be69-679150d34193
FreeSWITCH-Hostname: MiWiFi-R3D-srv
FreeSWITCH-Switchname: MiWiFi-R3D-srv
FreeSWITCH-IPv4: 192.168.31.164
FreeSWITCH-IPv6: ::1
Event-Date-Local: 2017-12-10 11:30:32
Event-Date-GMT: Sun, 10 Dec 2017 03:30:32 GMT
Event-Date-Timestamp: 1512876632835590
Event-Calling-File: mod_fun_asr.cpp
Event-Calling-Function: OnResultDataRecved
Event-Calling-Line-Number: 55
Event-Sequence: 914
ASR-Response: {"is_final":false,"mode":"2pass-offline","stamp_sents":[{"end":3875,"punc":"","start":2570,"text_seg":"都 怎 么 可 能 是","ts_list":[[2570,2770],[2770,3050],[3050,3290],[3290,3510],[3510,3670],[3670,3875]]}],"text":"都怎么可能是","timestamp":"[[2570,2770],[2770,3050],[3050,3290],[3290,3510],[3510,3670],[3670,3875]]","wav_name":"asr"}
Channel: sofia/external/linphone@192.168.31.210
```

​	Http接口 post方式 json数据格式：

​	

```
{"call_info":{"call_id": "c8379b9a-2c21-41c7-9a65-3636a29e2fbe","caller": "955555","callee": "4000"},"asr_result": "安","asr_type": "update_asr"}
```

​	
