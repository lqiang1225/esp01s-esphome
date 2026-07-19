# ESP-01S + TCRT5000 红外循迹传感器 - 入门验证测试

## 硬件接线 (方案A：单传感器)

```
TCRT5000        ESP-01S(烧录器)
-------------------------------
VCC  ----------  3.3V
GND  ----------  GND
D0   ----------  GPIO0
```

### 关于电源
ESP-01S 瞬间电流可达 300mA，烧录器（USB转串口模块）的 3.3V 输出通常可以驱动，
但如果观察到不稳定（频繁复位、WiFi连不上），请用独立 3.3V 稳压模块供电，
将 GND 与烧录器共地。

### 烧录器连接 (COM6)
ESP-01S 插入 USB 转串口模块时注意方向：
  - 通常 ESP-01S 的 GPIO0 侧朝外
  - 烧录时：GPIO0 需接地 → 上电 → 进入下载模式
  - 运行时：断开 GPIO0 接地 → 重新上电

---

## 开发环境搭建

### 1. 安装 Python
从 https://www.python.org/downloads/ 下载 Python 3.11+。
安装时勾选 **"Add Python to PATH"**。

验证安装：打开终端执行 `python --version`

### 2. 安装 ESPHome
```bash
pip install esphome
```

验证安装：`esphome version`

---

## 首次编译与烧录

### 生成加密密钥（首次编译前）
```bash
esphome wizard esp01s_tcrt5000.yaml
```
按提示设置：
  - 设备名称：esp01s-tcrt5000
  - 平台：ESP8266
  - 板型：esp01_1m
  - WiFi 名称和密码
  - 选择 Encryption Key（自动生成）

或者手动编辑 `secrets.yaml`：
```yaml
wifi_ssid: "你的WiFi名称"
wifi_password: "你的WiFi密码"
api_encryption_key: ""
ota_password: "12345678"
```

### 编译
```bash
esphome compile esp01s_tcrt5000.yaml
```

### 烧录到 ESP-01S
**步骤：**
1. 将 ESP-01S 插入烧录器，确保 GPIO0 已接地
2. 插入 USB（上电），模块进入下载模式
3. 确认 COM6 出现在设备管理器中
4. 执行：
```bash
esphome upload --device COM6 esp01s_tcrt5000.yaml
```
5. 烧录完成后，断开 GPIO0 接地
6. 重新上电（拔插 USB），模块开始运行

---

## 验证测试

### 方法1：看串口日志
烧录后执行：
```bash
esphome logs esp01s_tcrt5000.yaml --device COM6
```

将 TCRT5000 传感器分别对准 **白纸** 和 **黑线**，日志会显示：
```
[text_sensor:XXX]: Sensor Status Text: "White Surface"
[text_sensor:XXX]: Sensor Status Text: "Black Line DETECTED"
```

### 方法2：Web 界面
ESP-01S 上电后会尝试连接 WiFi。
连上后在浏览器访问 http://esp01s-tcrt5000.local 可查看传感器实时状态。

如果 WiFi 连不上（比如没设密码），模块会开启 AP 热点：
  - SSID: `ESP01S-TCRT5000 AP`
  - Password: `12345678`
  连接后访问 http://192.168.4.1

---

## 项目文件说明

| 文件 | 说明 |
|------|------|
| `esp01s_tcrt5000.yaml` | ESPHome 主配置文件 |
| `secrets.yaml` | WiFi/API 密钥（不要提交到 Git） |
| `README.md` | 本文件 |

---

## 下一步方向

- 方案B：双传感器循迹（GPIO0 + GPIO2）
- 接入 Home Assistant（已有 API 和加密配置）
- OTA 无线更新固件（后续不用再插线）
- 换 ESP32 做多路模拟量循迹
