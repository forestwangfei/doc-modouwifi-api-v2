# 魔豆路由器 HTTP API 规格文档

## 基本说明

- 系统操作处于锁的状态下返回 code=－1
- 身份鉴权基于 cookies
- 除非明确标出，所有 API 都需要进行身份鉴权
- 在未登录状态下访问需要 auth 的 API 会返回 403 状态码
- 单位：流量的单位（kbps），磁盘容量单位（MB），时间单位（s）
- 所有 POST 的请求的返回值的格式都为 JSON

基础返回格式：

```js
{
  "code"    : 0,                  // 返回代码，类型为数字，0为成功，其他失败
  "msg"     : "hello"             // 可能存在的出错消息
}
```

## 登录

**不需要身份验证**

`POST /api/auth/login`

```js
{
  "password" : "your password"
}
``` 

## 重启

### 正常重启

`GET /api/system/reboot`

```js
{
  "code"          : 0,
  "msg"           : ""
}
```

### 重启进安全模式

`GET /api/system/safe_reboot`

```js
{
  "code"          : 0,
  "msg"           : ""
}
```

### 恢复出厂设置

`GET /api/system/reset_config`

## WAN 口设置

### 获取当前 WAN 口设置

`GET /api/wan/get_info`

```js
{
  "type"              : "STATIC",              // IP 地址获取的方式(DHCP, PPPOE, STATIC)
  "connect_type"      : "STATIC",              // 连接方式(DHCP, PPPOE, STATIC, AP_CLIENT)
  "ip"                : "192.168.1.12",
  "mask"              : "182.168.1.1",
  "gateway"           : "255.255.255.0",
  "dns1"              : "8.8.8.8",
  "dns2"              : "8.8.4.4",
  "mtu"               : 2,
  "mtu_negotiable"    : 0,                     // 如果当前是PPPOE，默认为协商模式: 0
  "stp"               : true,
  "account"           : "account",             // 如果当前是PPPOE
  "password"          : "password",            // 如果当前是PPPOE
  "macCloneEnabled"   : true,                  // 是否开启Macclone
  "macCloneMac"       : "40:6c:8f:2d:6c:3b",   // MAC CLONE mac
  "uptime"            : "22486"                // 路由器运行时间
}
```

### 获取客户端 MAC 地址（用于 MAC 地址克隆）

`GET /api/wan/clientmacaddr`

```js
{
  "macaddr"   : "40:6C:8F:2D:6C:3A"
}
```

### 获取dhcp设置

`GET /api/wan/get_info/dhcp`

```js
{
  "dns1"    : "8.8.8.8",
  "dns2"    : "8.8.4.4",
  "mtu"     : 2,
  "stp"     : true
}
```

### 获取 PPPoE 设置

`GET /api/wan/get_info/pppoe`

```js
{
  "account"         : "account",
  "password"        : "password",
  "pppoe_method"    : "KeepAlive",    // 连接模式(KeepAlive, OnDemand, Manual)
  "pedial_period"   : 60,             // 连接断开xx秒后尝试重拨,单位(秒) 当前KeepAlive
  "idle_time"       : 5,              // 无流量时xx分钟后断开,单位(分) 当前OnDemand
  "status"          : 0               // -1: PPPoE暂时无状态；
                                      // 0: 连接已成功；
                                      // 1: 用户名／密码错误；
                                      // 2: 连接已断开;
                                      // 3: 不允许本帐户在此时间登录;
                                      // 4: 帐户已禁用;
                                      // 5: 密码已过期;
                                      // 6: 帐户没有远程访问权限;
                                      // 7: 未知错误.
}
```

### 获取静态 IP 设置

`GET /api/wan/get_info/static`

```js
{
  "ip"              : "192.168.1.12",
  "mask"            : "255.255.255.0",
  "gateway"         : "182.168.1.1",
  "dns1"            : "8.8.8.8",
  "dns2"            : "8.8.4.4",
  "mtu"             : 2,
  "stp"             : true
}
```

### 设置 WAN 口连接方式

`POST /api/wan/set_config`

post data:

```js
{
  "type"                  : "STATIC",             // IP 地址获取的方式(DHCP, PPPOE, STATIC)
  "connect_type"          : "STATIC",             // 连接方式(DHCP, PPPOE, STATIC, AP_CLIENT)
  "ip"                    : "192.168.1.12",
  "mask"                  : "255.255.255.0",
  "gateway"               : "192.168.1.1",
  "dns1"                  : "8.8.8.8",
  "dns2"                  : "8.8.4.4",
  "mtu"                   : 2,
  "stp"                   : true,
  "account"               : "account",            // 如果当前是PPPOE
  "password"              : "password",           // 如果当前是PPPOE
  "pppoe_method"          : "KeepAlive",          // 连接模式(KeepAlive, OnDemand, Manual)
  "pedial_period"         : 60,                   // 连接断开xx秒后尝试重拨,单位(秒) 当前KeepAlive
  "idle_time"             : 5,                    // 无流量时xx分钟后断开,单位(分) 当前OnDemand
  "macCloneEnabled"       : true,                 // 是否开启 Macclone
  "macCloneMac"           : "40:6c:8f:2d:6c:3b"   // MAC CLONE mac
}
```

return data:

```js
{
  "code"  : 0,          // (0->设置成功，1-> 正在设置，-1 ->已有全局设置锁)
  "msg"   : "xx"
}
```

### 检测互联网连通状态

`GET /api/wan/is_internet_available`

```js
{
  "code"    : 0    // 检测外网是否可以连接到互联网 (0, 正常连接互联网; 1, 不能正常连接; 
                                                    2, 解析域名失败; 3, 网路状况不佳, 比如下载导致的)
  "msg"     : "error message"   // 错误消息, 如果有的话.
}
```

### 获取 WAN 口自定义的DNS

`GET /api/wan/custom_dns/get`

return: 

```js
{
  "code"        : 0,                    // 返回码，0正常，非0出错
  "dns1"        : "8.8.8.8",            // 自定义DNS1
  "dns2"        : "8.8.4.4"             // 自定义DNS2
}
```

### 设置 WAN 口自定义DNS

`POST /api/wan/custom_dns/set`

```js
{
  "dns1"        : "8.8.8.8",            // 自定义DNS1
  "dns2"        : "8.8.4.4"             // 自定义DNS2
}
```

## WiFi 设置

### 取得 WiFi 的配置信息

`GET /api/wifi/get_config`

```js
{
  "2g":
    {
      "enabled"           : true,                   // 2.4g开关    RadioOff
      "ssid"              : "ssid1",                // 名称 SSID1(长度1-32字符)
      "broadcastssid"     : true,                   // 是否广播SSID
      "security_mode"     : "WPAPSKWPA2PSK",        // Security Mode
      "encrypt"           : "TKIP",                 // WPA Algorithms(TKIP,AES ,TKIPAES)  EncrypType
      "password"          : "12345678",             // 密码 AuthMode（长度8-64字符）
      "power"             : 20,                     // 无线信号功率	TXPower(0~1mw,4~2mw,7~5mw,10~10mw,13~13mw,16~39mw,19~79mw,20~100mw,22~158mw)
      "channel"           : 6,                      // 信道 Channel
      "net_type"          : 9,                      // 网络模式 WirelessMode (0,1,4,6,9,)
      "band_width_mode"   : 0,                      // 频道带宽 (0~自动，1~20HZ，2~40HZ)
      "mac"               : "28-2c-b2-97-82-39",    // mac地址 命令行ifconfig
      "beacon"            : 40,                     // Beacon时槽 BeaconPeriod (20~1024)
      "apsd_enabled"      : true,                   // APSD开关 APSDCapable
      "ap_enabled"        : true,                   // AP隔离开关 NoForwarding
      "shortgi_enabled"   : true,                   // short GI开关 HT_GI
      "wmm_enabled"       : true                    // 多媒体优先WMM开关 WmmCapable
    },
  "5g":
    {
      "enabled"           : true,                   // 5g开关
      "ssid"              : "ssid1",                // 名称
      "broadcastssid"     : true,                   // 是否广播SSID
      "security_mode"     : "WPAPSKWPA2PSK"         // Security Mode
      "encrypt"           : "TKIP",                 // WPA Algorithms(TKIP,AES ,TKIPAES) EncrypType
      "password"          : "12345678",             // 密码
      "power"             : 20,                     // 无线信号功率
      "channel"           : 14,                     // 信道
      "net_type"          : 14,                     // 网络模式 (2,8,14,15)
      "band_width_mode"   : 0,                      // 频道带宽 (0~自动，1~20HZ，2~40HZ，3~80HZ)
      "mac"               : "28-2c-b2-97-82-39",    // MAC 地址
      "beacon"            : 40,                     // Beacon 时槽
      "apsd_enabled"      : true,                   // APSD 开关
      "ap_enabled"        : true,                   // AP隔离开关
      "shortgi_enabled"   : true,                   // short GI 开关
      "wmm_enabled"       : true,                   // 多媒体优先WMM开关
      "same_as_2g"        : true                    // 使用与2.4g相同的设置
                                                    //（包含：无线名称，加密方式，加密算法，密码，传输功率，
                                                    // Beacon时槽，APSD，AP隔离，Short GI
                                                    // 多媒体优先WMM，无线广播）
    }
}
```

### 设置 WiFi（非阻塞）

`POST /api/wifi/set_config`

```js
{
  "2g":
    {
      "enabled"           : true,         // 2.4g开关 (true|false)
      "ssid"              : "ssid1",      // 名称(长度1-32字符) (any string)
      "broadcastssid"     : true,         // 是否广播SSID
      "security_mode"     : "Disable",    // Security Mode (Disable,WPAPSK,WPA2PSK,WPAPSKWPA2PSK)
      "encrypt"           : "TKIP",       // WPA Algorithms EncrypType
                                          // (NONE<>Disable,   TKIP<>WPA(2)PSK,
                                          // AES<>WPA(2)PSK ,   TKIPAES<>WPA(2)PSK)
      "password"          : "12345678",   // 密码(长度8-64字符) (any string)
      "power"             : 20,           // 无线信号功率 (0~1mw,4~2mw,7~5mw,10~10mw,13~13mw,16~39mw,19~79mw,20~100mw,22~158mw)
      "channel"           : 0,            // 哪个信道 (0)
                                          // {'name': '自动选择', 'value': 0},
                                          // {'name': '2412MHz (Channel 1)', 'value': 1},
                                          // {'name': '2417MHz (Channel 2)', 'value': 2},
                                          // {'name': '2422MHz (Channel 3)', 'value': 3},
                                          // {'name': '2427MHz (Channel 4)', 'value': 4},
                                          // {'name': '2432MHz (Channel 5)', 'value': 5},
                                          // {'name': '2437MHz (Channel 6)', 'value': 6},
                                          // {'name': '2442MHz (Channel 7)', 'value': 7},
                                          // {'name': '2447MHz (Channel 8)', 'value': 8},
                                          // {'name': '2452MHz (Channel 9)', 'value': 9},
                                          // {'name': '2457MHz (Channel 10)', 'value': 10},
                                          // {'name': '2462MHz (Channel 11)', 'value': 11},
                                          // {'name': '2467MHz (Channel 12)', 'value': 12},
                                          // {'name': '2472MHz (Channel 13)', 'value': 13}

      "net_type"          : 9,            // 网络模式 (0,1,4,6,9,)
                                          // 2G: 9
                                          // 0: legacy 11b/g mixed
                                          // 1: legacy 11B only
                                          // 4: legacy 11G only
                                          // 6: 11N only
                                          // 9: 11BGN mixed

      "band_width_mode"   : 1,            // 频道带宽(0~自动，1~20HZ，2~40HZ) 
      "beacon"            : 40,           // Beacon时槽 (20~1024)
      "apsd_enabled"      : true,         // APSD开关 (true|false)
      "ap_enabled"        : true,         // AP隔离开关 (true|false)
      "shortgi_enabled"   : true,         // short GI开关 (true|false)
      "wmm_enabled"       : true          // 多媒体优先WMM开关 (true|fales)
    },
  "5g":
    {
      "enabled"           : true,         // 5g开关
      "ssid"              : "ssid1",      // 名称(长度1-32字符)
      "broadcastssid"     : true,         // 是否广播SSID
      "security_mode"     : "Disable",    // Security Mode(Disable,WPAPSK,WPA2PSK,WPAPSKWPA2PSK)
      "encrypt"           : "TKIP",       // WPA Algorithms(TKIP,AES ,TKIPAES)
                                          // (NONE<>Disable, TKIP<>WPA(2)PSK,
                                          // AES<>WPA(2)PSK, TKIPAES<>WPA(2)PSK)
      "password"          : "12345678",   // 密码(长度8-64字符)
      "power"             : 20,           // 无线信号功率 (100,90,60,30,15,0)
      "channel"           : 0,            // 哪个信道
                                          // {'name': '自动选择', 'value': 0},
                                          // {'name': '5180MHz (Channel 36)', 'value': 36},
                                          // {'name': '5200MHz (Channel 40)', 'value': 40},
                                          // {'name': '5220MHz (Channel 44)', 'value': 44},
                                          // {'name': '5240MHz (Channel 48)', 'value': 48},
                                          // {'name': '5260MHz (Channel 52)', 'value': 52},
                                          // {'name': '5280MHz (Channel 56)', 'value': 56},
                                          // {'name': '5300MHz (Channel 60)', 'value': 60},
                                          // {'name': '5320MHz (Channel 64)', 'value': 64},
                                          // {'name': '5745MHz (Channel 149)', 'value': 149},
                                          // {'name': '5765MHz (Channel 153)', 'value': 153},
                                          // {'name': '5785MHz (Channel 157)', 'value': 157},
                                          // {'name': '5805MHz (Channel 161)', 'value': 161},
                                          // {'name': '5825MHz (Channel 165)', 'value': 165}


      "net_type"          : 14,           // 网络模式 (2,8,14,15)
                                          // 5G 14
                                          // 2: legacy 11A only
                                          // 8: 11AN mixed
                                          // 14: 11A/AN/AC mixed 5G band only
                                          // 15: 11 AN/AC mixed 5G band only

      "band_width_mode"   : 0,            // 频道带宽 (0~自动，1~20HZ，2~40HZ，3~80HZ)
      "beacon"            : 40,           // Beacon时槽
      "apsd_enabled"      : true,         // APSD开关
      "ap_enabled"        : true,         // AP隔离开关
      "shortgi_enabled"   : true,         // short GI开关
      "wmm_enabled"       : true,         // 多媒体优先WMM开关
      "same_as_2g"        : true          // 使用与2.4g相同的设置（包含：无线名称，加密方式，
                                          // 加密算法，密码，传输功率，Beacon时槽，APSD，AP隔离，
                                          // Short GI，多媒体优先WMM，无线广播）
    }
}
```

`GET /api/wifi/check_set`

```js
{
  "code"                  : 0,            // (0->设置成功，1-> 正在设置，-1 ->已有全局设置锁)
  "msg"                   : "xx"
}
```

### 无线网络是否已打开

`GET /api/wifi/is_enabled`

```js
{
  "is_enabled"            : bool,         // 是否已经打开了wifi
}
```

### 请求打开wps

`GET /api/wifi/start_wps`
```js
{
  "code"            : 0,         // 0为成功，其他为失败
}
```


## LAN 口

### 获取 LAN 口设置

`GET /api/lan/get_lan_config`

```js
{
  "ip"                    : "192.168.1.2",                  // ip
  "net_mask"              : "255.255.255.0",                // 子网掩码
  "dhcp_enabled"          : true,                           // dhcp开关
  "ipaddr_start"          : "4.4.4.4",                      // IP地址开始段
  "ipaddr_end"            : "4.4.4.4",                      // IP地址结束段
  "gateway"               : "192.168.1.0",                  // 网关
  "dhcp_net_mask"         : "255.255.255.0",                // dhcp子网掩码
  "dns1Method"            : "手动",                         // dns1设置方式（手动，自动）
  "dns2Method"            : "自动",                         // dns2设置方式（手动，自动）
  "dns1"                  : "8.8.8.8",                      // 首选dns
  "dns2"                  : "",                             // 备用dns
  "time"                  : 381,                            // 地址租期
  "mac"                   : "28-2c-b2-97-82-39"             // mac地址
}
```

### 修改 LAN 口设置

`POST /api/lan/set_lan_config`

```js
{
  "ip"                    : "192.168.1.2",                  // ip
  "net_mask"              : "255.255.255.0",                // 子网掩码
  "dhcp_enabled"          : true,                           // dhcp开关
  "ipaddr_start"          : "4.4.4.4",                      // IP地址开始段
  "ipaddr_end"            : "4.4.4.4",                      // IP地址结束段
  "gateway"               : "192.168.1.0",                  // 网关
  "dhcp_net_mask"         : "255.255.255.0",                // dhcp子网掩码
  "dns1Method"            : "手动",                         // dns1设置方式（手动，自动）
  "dns2Method"            : "自动",                         // dns2设置方式（手动，自动）
  "dns1"                  : "8.8.8.8",                      // 首选dns
  "dns2"                  : "",                             // 备用dns
  "time"                  : 381,                            // 地址租期
  "mac"                   : "28-2c-b2-97-82-39"             // mac地址
}
```

`GET /api/lan/check_set`

```js
{
  "code"                  : 0,          // (0->设置成功，1-> 正在设置，2-> 设置失败)
  "msg"                   : "xx"
}
```

### 获取物理连接情况

`GET /api/system/get_cable_connection`

```js
{
  "wan"                   : true,             // wan口是否有物理连接
  "lan1"                  : true,             // wan口是否有物理连接
  "lan2"                  : true,             // wan口是否有物理连接
  "usb"                   : true              // usb是否有连接
}
```


