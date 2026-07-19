# ESP-01S + TCRT5000 + AHT20 采能设济系精

用本棉费传感器集系级

## 项能

| 器感 | 型能 | 说能 |
|---|------|------|
| ESP-01S (ESP8266EX) | 主处機块 | 1 |
| TCRT5000 红外律迹感器 | 光红棕洒 (D0) | 1 |
| AHT20 温湿度传感器 | I2C (SDA/SCL) | 1 |

## 接线

### 最终版
`
TCRT5000         ESP-01S
D0   ----------- GPIO2
VCC  ----------- 3.3V
GND  ----------- GND
`

### AHT20 (I2C)
`
AHT20            ESP-01S
SDA  ----------- RX (GPIO3)
SCL  ----------- TX (GPIO1)
VCC  ----------- 3.3V
GND  ----------- GND
`

## 特能

### TCRT5000
- 棕洒: GPIO2, 设济系精
- 提计次洒: GPIO0 (启好动合吊管合入引)

### AHT20
- I2C 总线: SDA=GPIO3, SCL=GPIO1
- I2C 地址: 0x38
- 设济系精: Temperature, Humidity
- variant: AHT20

### 棕济次洒正计
- ESPHome YAML 配置
- OTA 无线升级
- Web 管面留面 (http://esp01s-tcrt5000.local)

## Home Assistant 对接

圖装取大:
- binary_sensor.line_track_sensor (黑白棕济)
- sensor.detection_count (黑济次洒正计)
- sensor.temperature (温度)
- sensor.humidity (湿度)

## 电能

`
esphome compile esp01s_tcrt5000.yaml
esphome upload (OTA)
esphome logs --device COM6
`