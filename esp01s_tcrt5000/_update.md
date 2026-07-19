<title>ESP-01S + TCRT5000 红外循迹传感器 实验报告</title>

# ESP-01S + TCRT5000 红外循迹传感器 实验报告

## 实验概述

使用 ESP-01S(ESP8266) 模块驱动 TCRT5000 红外循迹传感器,通过 ESPHome 平台  
开发固件,实现黑白线检测、串口日志输出、WiFi 联网和 Web 管理界面。

**开发工具:** ESPHome 2026.6.5(基于 Python 3.12.3)  
**编程方式:** YAML 配置(ESPHome 原生)

---

## 硬件清单

| 器件 | 型号/规格 | 数量 |
|-|-|-|
| WiFi 模块 | ESP-01S(ESP8266EX, 1MB Flash) | 1 |
| 红外循迹传感器 | TCRT5000 模块(带 LM393 比较器) | 1 |
| USB 转串口烧录器 | CH340/CP2102 | 1 |
| 杜邦线 | 母对母 | 若干 |

---

## 接线图(最终版)

## \`

TCRT5000          ESP-01S

VCC  ----------- 3.3V  
GND  ----------- GND  
D0   ----------- GPIO2  
\`

### 烧录时额外接线(临时)

`GPIO0 ----------- GND    (进入下载模式,烧录完成后断开)VCC   ----------- 3.3VCH_PD/EN -------- 3.3V   (必须接高,否则模块不工作)`

### 烧录器与 ESP-01S 接线

`烧录器 TX ------ ESP-01S RX烧录器 RX ------ ESP-01S TX烧录器 3.3V ---- ESP-01S VCC, CH_PD(EN)烧录器 GND ----- ESP-01S GND`

> 注意:烧录器和 ESP-01S 的 TX/RX 必须交叉连接。

---

## 开发环境搭建

### 安装 Python 3.12.3

\`powershell  
Invoke-WebRequest -Uri "https://www.python.org/ftp/python/3.12.3/python-3.12.3-amd64.exe"  
-OutFile "C:\Users\LQIANG\~1\AppData\Local\Temp\python-installer.exe" -UseBasicParsing -TimeoutSec 60

Start-Process -FilePath "C:\Users\LQIANG\~1\AppData\Local\Temp\python-installer.exe"  
-ArgumentList "/quiet InstallAllUsers=0 PrependPath=1 TargetDir=C:\Users\lqiang1225\AppData\Local\Programs\Python\Python312"  
-Wait -NoNewWindow  
\`

### 安装 ESPHome(清华镜像源)

`powershellpip install esphome -i https://pypi.tuna.tsinghua.edu.cn/simple --trusted-host pypi.tuna.tsinghua.edu.cn`

### 验证

\`powershell  
esphome version

# 输出: Version: 2026.6.5

\`

---

## 项目文件结构

`esp01s_tcrt5000/├── esp01s_tcrt5000.yaml    # ESPHome 主配置文件├── secrets.yaml             # WiFi/API 密钥├── 实验报告.md              # 本文档└── .esphome/                # 编译缓存(自动生成)`

---

## 配置文件

### esp01s_tcrt5000.yaml

\`yaml  
esphome:  
name: esp01s-tcrt5000  
friendly_name: ESP01S_TCRT5000

esp8266:  
board: esp01_1m

wifi:  
ssid: !secret wifi_ssid  
password: !secret wifi_password  
ap:  
ssid: "ESP01S-TCRT5000 AP"  
password: "12345678"

captive_portal:

logger:  
baud_rate: 115200  
level: INFO

api:  
encryption:  
key: !secret api_encryption_key

ota:

- platform: esphome  
password: !secret ota_password

web_server:  
port: 80

binary_sensor:

- platform: gpio  
name: "Line Track Sensor"  
id: "sensor_track"  
pin:  
number: GPIO2  
mode: INPUT  
device_class: presence  
publish_initial_state: true  
on_state:  
then:  
\- if:  
condition:  
binary_sensor.is_on: sensor_track  
then:  
\- logger.log:  
format: "Sensor: White Surface"  
level: INFO  
else:  
\- logger.log:  
format: "Sensor: Black Line DETECTED"  
level: INFO  
\`

### secrets.yaml

`yamlwifi_ssid: "ZTE-456KKE-2.4G"wifi_password: "qiang6621"api_encryption_key: "vHz9Fpdu2Fhbeaa2NIOpLI1vVxREBclDnF+Bt73YwZA="ota_password: "12345678"`

---

## 编译与烧录

### 常用命令

\`powershell

# 校验配置

esphome config C:\path\to\esp01s_tcrt5000.yaml

# 编译

esphome compile C:\path\to\esp01s_tcrt5000.yaml

# 清除缓存强制重编

Remove-Item -Recurse -Force .esphome -ErrorAction SilentlyContinue  
esphome compile C:\path\to\esp01s_tcrt5000.yaml

# 烧录(COM6)

esphome upload --device COM6 C:\path\to\esp01s_tcrt5000.yaml

# 查看日志

esphome logs C:\path\to\esp01s_tcrt5000.yaml --device COM6  
\`

### 烧录步骤

1. GPIO0 接地
2. 插入 USB 上电(进入下载模式)
3. 执行 esphome upload
4. 烧录完成,断开 GPIO0 接地
5. 重新上电,进入运行模式

---

## 测试结果

### 1. 传感器数据

`[23:45:13.895][I][main:349]: Sensor: White Surface[23:45:15.748][I][main:353]: Sensor: Black Line DETECTED[23:45:16.392][I][main:349]: Sensor: White Surface`

白纸 -> White Surface, 黑线 -> Black Line DETECTED

### 2. WiFi 连接

成功连接路由器 ZTE-456KKE-2.4G,同局域网设备可正常访问。

### 3. Web 界面

浏览器访问 [http://esp01s-tcrt5000.local](http://esp01s-tcrt5000.local),传感器变化时页面实时更新。

### 4. 板载蓝灯

GPIO2 同时控制板载蓝色 LED,检测到黑线时亮起,作为物理状态指示。

---

## 经验教训

1. **GPIO0 不可接 IO 设备** - 上电拉低会进入下载模式
2. **改用 GPIO2** - 蓝灯同步指示(亮=黑线)
3. **CH_PD/EN 必须接 3.3V** - 否则芯片不工作
4. **烧录后端口可能锁死** - 拔插 USB 释放 COM 口
5. **国内网络用清华镜像源** - pip install -i [https://pypi.tuna.tsinghua.edu.cn/simple](https://pypi.tuna.tsinghua.edu.cn/simple)
6. **上电前传感器放浅色表面** - 确保 GPIO2 启动时为高电平

---

## 后续方向

- **方案B:双传感器循迹** - GPIO2 + GPIO0 各接一个 TCRT5000
- **接入 Home Assistant** - API 加密已预置
- **OTA 无线更新** - 无需插串口线
- **换 ESP32** - 支持多路模拟量循迹

---

## 5. 检测次数累计

在 esp01s_tcrt5000.yaml 中新增 globals 和 sensor 组件实现黑线检测次数的自动累计:

\`yaml  
globals:

- id: detection_count  
type: int  
restore_value: yes  
initial_value: "0"

sensor:

- platform: template  
name: "Detection Count"  
id: detection_count_sensor  
lambda: |-  
return id(detection_count);  
update_interval: 1s  
unit_of_measurement: "times"  
icon: "mdi:counter"  
\`

每次检测到黑线自动 +1,断电后数值保存。

---

## 6. 网页端 OTA 升级

通过 Web 界面无线升级固件,无需串口线。

1. 终端执行 esphome compile 编译固件
2. 编译后在 .pioenvs 目录生成 firmware.bin
3. 浏览器打开 [http://esp01s-tcrt5000.local](http://esp01s-tcrt5000.local)
4. 点击 Choose File 选择 firmware.bin
5. 点击 Update 等待完成,模块自动重启

---

## 7. Home Assistant 对接

HA 中安装 ESPHome 集成后自动发现。  
输入加密密钥后连接,出现两个实体:

- binary_sensor.line_track_sensor — 黑白检测
- sensor.detection_count — 累计次数

---

## 8. AHT20 温湿度传感器扩展

### 硬件接线

AHT20 I2C,RX/TX I2C:

`AHT20            ESP-01SVCC  ----------- 3.3VGND  ----------- GNDSDA  ----------- RX (GPIO3)SCL  ----------- TX (GPIO1)`

### YAML

\`yaml  
logger:  
baud_rate: 0

i2c:  
sda: GPIO3  
scl: GPIO1  
scan: true

sensor:

- platform: aht10  
temperature:  
name: Temperature  
humidity:  
name: Humidity  
update_interval: 10s  
\`

### HA 实体

- sensor.temperature
- sensor.humidity

### 诊断信息

如果 AHT20 未响应,请检查:

1. SDA/SCL 接线是否正确(SDA→RX,SCL→TX)
2. 上拉电阻是否缺少(SDA/SCL 对 3.3V 各加 4.7KΩ)
3. AHT20 供电是否稳定

添加 text_sensor 和 variant: AHT20 后正常读取数据。
