# Home Assistant 智能家居配置指南

## 目录
- [项目简介](#项目简介)
- [系统要求](#系统要求)
- [安装指南](#安装指南)
- [基础配置](#基础配置)
- [进阶功能](#进阶功能)
- [常见问题](#常见问题)
- [维护指南](#维护指南)
- [参考资源](#参考资源)

## 项目简介

Home Assistant 是一个功能强大的开源家庭自动化平台，支持：
- 3000+ 集成组件
- 可视化仪表盘
- 强大的自动化功能
- 本地控制优先
- 完全的隐私保护
- 活跃的社区支持

### 核心优势
- 完全本地化运行，不依赖云服务
- 支持几乎所有主流智能家居设备
- 强大的自动化和场景控制
- 丰富的自定义选项
- 完善的 API 接口

## 系统要求

### 最低配置
- CPU: 1.5GHz 双核处理器
- 内存: 2GB RAM
- 存储: 32GB
- 网络: 有线或无线连接

### 推荐配置
- CPU: 2GHz 四核处理器
- 内存: 4GB RAM
- 存储: 64GB SSD
- 网络: 千兆有线连接

### 支持的操作系统
- Home Assistant OS (推荐)
- Linux (Debian/Ubuntu)
- Windows 10/11
- macOS
- Docker 环境

### 支持的硬件平台
- 树莓派（所有型号）
- Intel NUC
- 通用 x86-64 设备
- 虚拟机环境
- ODROID
- 群晖 NAS

## 安装指南

### 方式一：Home Assistant OS（新手推荐）

1. 下载系统镜像
   ```bash
   # 访问下载页面
   https://www.home-assistant.io/installation/
   ```

2. 系统安装
   - 使用 balenaEtcher 写入镜像
   - 首次启动等待 20 分钟左右
   - 访问 http://homeassistant.local:8123

3. 初始设置
   - 创建管理员账户
   - 设置时区和位置
   - 配置网络

### 方式二：Docker 安装（适合高级用户）

1. 安装 Docker
   ```bash
   curl -fsSL https://get.docker.com | sh
   ```

2. 运行容器
   ```bash
   docker run -d \
     --name homeassistant \
     --privileged \
     --restart=unless-stopped \
     -e TZ=Asia/Shanghai \
     -v /PATH_TO_YOUR_CONFIG:/config \
     --network=host \
     ghcr.io/home-assistant/home-assistant:stable
   ```

### 方式三：Python 虚拟环境安装

1. 安装依赖
   ```bash
   sudo apt-get update
   sudo apt-get install -y python3 python3-dev python3-venv python3-pip
   ```

2. 创建虚拟环境
   ```bash
   python3 -m venv homeassistant
   cd homeassistant
   source bin/activate
   ```

3. 安装 Home Assistant
   ```bash
   pip3 install homeassistant
   ```

### 准备工作
1. 硬件准备
   - 选择合适的硬件平台
   - 准备存储介质（SD卡/SSD）
   - 准备网线或确保WiFi可用

2. 网络准备
   - 确保有稳定的互联网连接
   - 准备一个固定的IP地址（推荐）
   - 确保路由器支持DHCP

3. 工具准备
   - 下载 balenaEtcher（镜像写入工具）
   - 准备 SSH 客户端（可选）
   - 准备文本编辑器

## 基础配置

### 1. 核心配置文件

`configuration.yaml` 示例：
```yaml
# 基础设置
homeassistant:
  name: My Smart Home
  latitude: !secret latitude
  longitude: !secret longitude
  elevation: !secret elevation
  unit_system: metric
  time_zone: Asia/Shanghai
  currency: CNY

# 核心组件
default_config:
frontend:
  themes: !include_dir_merge_named themes
  
# 自动化和场景
automation: !include automations.yaml
script: !include scripts.yaml
scene: !include scenes.yaml

# 自定义组件
sensor: !include_dir_merge_list sensors
switch: !include_dir_merge_list switches
```

### 2. 网络配置

```yaml
# 网络设置
http:
  ssl_certificate: !secret ssl_certificate
  ssl_key: !secret ssl_key
  use_x_forwarded_for: true
  trusted_proxies:
    - 127.0.0.1
    - ::1
```

### 3. 用户管理

```yaml
# 用户配置
person:
  - name: 管理员
    id: admin
    device_trackers:
      - device_tracker.phone_admin
```

### 4. 设备接入指南

#### 小米设备接入
```yaml
# configuration.yaml
xiaomi_miio:
  interface: 0.0.0.0
  devices:
    - host: 192.168.1.x
      token: !secret xiaomi_token
      model: zhimi.airpurifier.v6
```

#### 涂鸦设备接入
```yaml
# configuration.yaml
tuya:
  username: !secret tuya_username
  password: !secret tuya_password
  country_code: 86
  platform: smart_life
```

#### ESPHome设备接入
```yaml
# configuration.yaml
esphome:
  - platform: mqtt
    name: "客厅温湿度"
    device_class: sensor
```

## 进阶功能

### 1. MQTT 集成

```yaml
# MQTT 配置
mqtt:
  broker: !secret mqtt_broker
  port: !secret mqtt_port
  username: !secret mqtt_username
  password: !secret mqtt_password
  discovery: true
  discovery_prefix: homeassistant
```

### 2. 自动化示例

```yaml
# 日落自动开灯
automation:
  - alias: "日落开灯"
    trigger:
      platform: sun
      event: sunset
      offset: "-00:30:00"
    action:
      - service: light.turn_on
        target:
          entity_id: 
            - light.living_room
            - light.hallway
        data:
          brightness_pct: 70
          transition: 5
```

### 3. 自定义仪表盘

```yaml
# Lovelace UI 配置
lovelace:
  mode: yaml
  resources:
    - url: /hacsfiles/mini-graph-card/mini-graph-card-bundle.js
      type: module
```

### 4. 备份策略

1. 自动备份
```yaml
# 自动备份配置
automation:
  - alias: "每周备份"
    trigger:
      platform: time
      at: "03:00:00"
    condition:
      condition: time
      weekday:
        - mon
    action:
      - service: backup.create
        data:
          name: "weekly_backup_{{ now().strftime('%Y%m%d') }}"
```

2. 重要文件备份清单：
   - configuration.yaml
   - secrets.yaml
   - automations.yaml
   - scripts.yaml
   - .storage 目录
   - custom_components 目录

### 5. 场景示例

```yaml
# 回家场景
scene:
  - name: 回家模式
    entities:
      light.living_room:
        state: on
        brightness_pct: 80
      climate.living_room:
        state: heat
        temperature: 24
      media_player.living_room:
        state: on
        source: 网易云音乐
```

### 6. 自动化进阶

#### 条件判断示例
```yaml
automation:
  - alias: "空调智能控制"
    trigger:
      - platform: numeric_state
        entity_id: sensor.temperature
        above: 26
    condition:
      condition: and
      conditions:
        - condition: time
          after: '08:00:00'
          before: '22:00:00'
        - condition: state
          entity_id: binary_sensor.someone_home
          state: 'on'
    action:
      - service: climate.set_temperature
        target:
          entity_id: climate.living_room
        data:
          temperature: 24
```

#### 模板使用示例
```yaml
sensor:
  - platform: template
    sensors:
      home_status:
        friendly_name: "家庭状态"
        value_template: >
          {% if is_state('binary_sensor.someone_home', 'on') %}
            有人在家
          {% else %}
            外出
          {% endif %}
```

## 常见问题

### 1. 连接问题
- 检查网络连接
- 验证防火墙设置
- 确认端口是否开放
- DNS 解析是否正确

### 2. 性能优化
- 定期清理数据库
- 优化自动化逻辑
- 减少不必要的集成
- 使用高效的存储设备

### 3. 安全加固
- 启用 2FA 认证
- 使用强密码
- 定期更新系统
- 限制访问范围

### 4. 数据库维护
- 定期清理历史数据
  ```bash
  # 清理90天前的数据
  recorder:
    purge_keep_days: 90
    auto_purge: true
  ```
- 优化数据库性能
  ```bash
  # 数据库优化配置
  recorder:
    db_url: mysql://user:password@SERVER_IP/DB_NAME
    commit_interval: 1
  ```

### 5. Z-Wave/Zigbee设备问题
- 检查USB设备识别
- 确认协调器固件版本
- 排查设备配对失败原因
- 处理设备离线问题

## 维护指南

### 1. 日常维护
- 检查系统日志
- 监控系统资源
- 验证自动化运行
- 测试关键功能

### 2. 更新流程
1. 备份当前系统
2. 检查更新日志
3. 更新系统
4. 验证功能正常

### 3. 故障排除
- 查看错误日志
- 检查配置文件
- 验证网络连接
- 测试硬件状态

### 4. 性能调优
- 使用 MariaDB/MySQL 替代 SQLite
- 优化历史数据存储
- 配置日志级别
- 使用缓存机制

### 5. 安全最佳实践
- 使用 HTTPS
- 配置反向代理
- 启用 IP 封禁
- 定期更新密码
- 使用 secrets.yaml 管理敏感信息

## 参考资源

### 1. 官方资源
- [Home Assistant 文档](https://www.home-assistant.io/docs/)
- [开发者文档](https://developers.home-assistant.io/)
- [蓝图库](https://www.home-assistant.io/blueprints/)

### 2. 社区资源
- [中文论坛](https://bbs.hassbian.com/)
- [GitHub 仓库](https://github.com/home-assistant)
- [Discord 频道](https://www.home-assistant.io/join-chat/)

### 3. 学习资源
- [视频教程](https://www.youtube.com/c/HomeAssistant)
- [示例配置](https://github.com/search?q=home+assistant+config)
- [常见问题解答](https://www.home-assistant.io/faq/)

## 版本控制

请定期检查并记录版本更新：
```
当前版本：2023.12.x
最低支持版本：2023.1.0
推荐版本：2023.12.x 或更高
```

## 许可说明

本配置指南采用 MIT 许可证。你可以自由使用、修改和分发，但需保留原始版权信息。

## 贡献指南

欢迎提供改进建议和贡献代码：
1. Fork 项目
2. 创建特性分支
3. 提交变更
4. 发起 Pull Request

## 联系方式

如需帮助，可通过以下方式获取支持：
- GitHub Issues
- 电子邮件：support@home-assistant.io
- 社区论坛：https://community.home-assistant.io/ 