```c
/*设置MQTT客户端配置信息，用户名，密码*/
AT+MQTTUSERCFG=0,1,"NULL","DN100_00000001","8772cb86fa67c6d961428453447750a2924793565909e764271903f9699b8445",0,0,""
    
OK

/*设置MQTT客户端ID*/
AT+MQTTCLIENTID=0,"DN100_00000001_0_0_2025092416"

OK
    
/***********************************上面两项初始化的时候就做，没网也可以进行*********************************************/

/*连接MQTT服务器*/
AT+MQTTCONN=0,"ec178aa6a7.st1.iotda-device.cn-east-3.myhuaweicloud.com",1883,1
+MQTTCONNECTED:0,1,"ec178aa6a7.st1.iotda-device.cn-east-3.myhuaweicloud.com","1883","",1

OK
  
/*如果有OTA通知的话，这个时候就会收到通知信息*/
+MQTTSUBRECV:0,"$oc/devices/DN100_00000000/sys/events/down",162,{"object_device_id":"DN100_00000000","services":[{"event_type":"version_query","service_id":"$ota","event_time":"20250710T024800Z","paras":null}]}

/*订阅下载主题*/
AT+MQTTSUB=0,"$oc/devices/DN100_00000001/sys/events/down",1

OK
    
/*上传设备数据*/
AT+MQTTPUBRAW=0,"$oc/devices/DN100_00000000/sys/properties/report",763,0,0

OK

>
{"services":[{"service_id":"DevInfo","properties":{"shitIR":0,"imuPitch":0,"imuRoll":0,"imuYaw":0,"waterLf":0,"waterLb":0,"waterRf":0,"waterRb":0,"thermalImg":0,"wearIR":0,"bagPressure":0,"fanPressure":0,"fanOutTemp":0,"waterInTemp":0,"waterOutTemp":0,"waterFlow":0,"fanCurrent":0,"pumpCurrent":0,"cleanTemp":0,"cleanLvl":0,"cleanTime":0,"windTemp":0,"windLvl":0,"windTime":0,"hotTemp":0,"hotLvl":0,"hotTime":0,"deodorLvl":0,"bagpreLvl":0,"shitNum":0,"weeNum":0,"turnNmu":0,"nurseNum":0,"sideleakNum":0,"filterLife":0,"pipeLife":0}}]}/*不加\r\n*/
busy p...
+MQTTPUB:OK
    
/*上报版本信息*/
AT+MQTTPUB=0,"$oc/devices/DN100_00000000/sys/events/up","{\"services\":[{\"service_id\":\"$ota\"\,\"event_type\":\"version_report\"\,\"paras\":{\"fw_version\":\"1.0\"}}]}",0,0

OK
+MQTTSUBRECV:0,"$oc/devices/68590ece7d33413cbac6c7c5_11111/sys/events/down",582,{"object_device_id":"68590ece7d33413cbac6c7c5_11111","services":[{"event_type":"firmware_upgrade","service_id":"$ota","event_time":"20250710T055429Z","paras":{"version":"2.0","url":"https://124.70.218.131:8943/iodm/dev/v2.0/upgradefile/applications/928f555068774817aa8010a996b21207/devices/68590ece7d33413cbac6c7c5_11111/packages/685b7386bd1c7a5b088e9498","file_size":30720,"file_name":"Newsensorboard.bin","access_token":"d2e880432ab35e18c4f8629663ce54f7f08e7b2e16efe03bf39ea5081f39b478","expires":86400,"sign":"71cfa4cfb93fce5c9677086ff7f5f2d786cfd2c6f27d4636978e1b9a8b2b12f5"}}]}


/*设置请求头*/
[10:54:01.117] AT+HTTPCHEAD=0
[10:54:01.126] 
[10:54:01.126] OK
[11:08:42.517] AT+HTTPCHEAD=19
[11:08:42.523] 
[11:08:42.523] OK
[11:08:42.523] 
[11:08:42.523] >
[11:08:49.667] Range: bytes=0-2047
[11:08:49.672] 
[11:08:49.672] OK
[11:09:32.167] AT+HTTPCHEAD=86
[11:09:32.173] 
[11:09:32.173] OK
[11:09:32.173] 
[11:09:32.173] >
[11:11:26.618] Authorization: Bearer d2e880432ab35e18c4f8629663ce54f7f08e7b2e16efe03bf39ea5081f39b478
[11:11:26.631] 
[11:11:26.631] OK
  
//直接下载所有固件
AT+HTTPCGET="https://124.70.218.131:8943/iodm/dev/v2.0/upgradefile/applications/babd3f246b06468dae06937576fe348a/devices/DN100_00000000/packages/6881c874bd1c7a5b0891f482"
+HTTPCGET:1762,***********************\r\n（其中的*代表很多16进制数）
+HTTPCGET:286,**********************\r\n（其中的*代表很多16进制数）
\r\n
OK\r\n

/*再次设置HTTP头*/
AT+HTTPCHEAD=0

OK

AT+HTTPCHEAD=22

OK

>
Range: bytes=2048-4095

busy p...

OK

AT+HTTPCHEAD=86

OK

>
Authorization: Bearer d2e880432ab35e18c4f8629663ce54f7f08e7b2e16efe03bf39ea5081f39b478
OK

/*再次发起下载请求*/
AT+HTTPCGET="https://124.70.218.131:8943/iodm/dev/v2.0/upgradefile/applications/928f555068774817aa8010a996b21207/devices/68590ece7d33413cbac6c7c5_11111/packages/685b7386bd1c7a5b088e9498"
+HTTPCGET:1110,***********************\r\n（其中的*代表很多16进制数）
+HTTPCGET:938,**********************\r\n（其中的*代表很多16进制数）
\r\n
OK\r\n

/*上报更新完毕信息*/
AT+MQTTPUB=0,"$oc/devices/68590ece7d33413cbac6c7c5_11111/sys/events/up","{\"services\":[{\"service_id\":\"$ota\"\,\"event_type\":\"upgrade_progress_report\"\,\"paras\":{\"result_code\":0\,\"version\":\"2.0\"}}]}",0,0

```
