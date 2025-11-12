```c
/* 关闭回显，收到两次响应 */
ATE0
ATE0

OK

/* 设置WIFI开启STA模式，响应OK */
AT+CWMODE=1,0

OK

/* 扫描WIFI信号，响应信号列表 */
AT+CWLAP
+CWLAP:(3,"deyintech",-27,"44:f7:70:b4:2a:ca",6,-1,-1,4,4,7,1)
+CWLAP:(0,"ESP_E6D1CD",-46,"f0:24:f9:e6:d1:cd",1,-1,-1,0,0,7,0)
+CWLAP:(3,"AWS",-49,"a4:39:b3:cd:10:42",6,-1,-1,4,4,7,1)
+CWLAP:(0,"ESP_E6D1ED",-51,"f0:24:f9:e6:d1:ed",1,-1,-1,0,0,7,0)
+CWLAP:(3,"Galaxy S20 5G",-51,"0a:e2:1e:a3:55:69",6,-1,-1,4,4,7,0)
+CWLAP:(3,"leaderobot",-53,"22:6e:6e:f2:6a:c2",6,-1,-1,4,4,7,1)
+CWLAP:(3,"DIRECT-XELUVImsYm",-60,"7a:46:5c:42:10:a5",6,-1,-1,4,4,6,1)
+CWLAP:(3,"DD-Office",-64,"00:dd:b6:15:bc:b0",1,-1,-1,4,4,7,0)
+CWLAP:(3,"DIRECT-54-HP M232 LaserJet",-66,"52:c2:e8:f6:2e:54",6,-1,-1,4,4,6,1)
+CWLAP:(3,"DIRECT-OmLAPTOP-BHBC8PV6mson",-66,"b6:b5:b6:ea:c1:3f",6,-1,-1,4,4,6,1)
+CWLAP:(3,"Hello",-69,"00:dd:b6:15:bc:b2",1,-1,-1,4,4,7,0)
+CWLAP:(3,"deyintech",-69,"44:f7:70:b4:2b:02",6,-1,-1,4,4,7,1)
+CWLAP:(4,"SWD-Office",-70,"00:dd:b6:15:bc:b1",1,-1,-1,4,4,7,0)
+CWLAP:(3,"IW23012100912",-85,"b6:8a:0a:d6:62:ed",1,-1,-1,5,3,3,0)

OK

/* 连接WIFI，响应WIFI CONNECTED */
AT+CWJAP="deyintech","zhimakaimen"
WIFI CONNECTED
WIFI GOT IP

OK

/* 如果输入了错误的密码，响应+CWJAP:2 */
AT+CWJAP="deyintech","afasdfsdaf"
+CWJAP:2
    
/* 断开连接 */
WIFI DISCONNECT
```
