# Home Assistant 设备配置与仪表盘设计

## 一、设备配置指南

### 1. 小米设备配置

```yaml
# configuration.yaml

# 1.1 小米网关与子设备
xiaomi_aqara:
  discovery_retry: 5
  gateways:
    - key: !secret xiaomi_gateway_key  # 在secrets.yaml中配置密钥

# 1.2 小米设备（Wi-Fi设备）
xiaomi_miio:
  interface: 0.0.0.0
  devices:
    # 空调伴侣
    - host: 192.168.1.x
      token: !secret xiaomi_ac_token
      name: living_room_ac_partner
      model: lumi.acpartner.v3

    # 智能插座
    - host: 192.168.1.x
      token: !secret xiaomi_plug_token
      name: bedroom_plug
      model: chuangmi.plug.v2

    # 空气净化器
    - host: 192.168.1.x
      token: !secret xiaomi_airpurifier_token
      name: living_room_purifier
      model: zhimi.airpurifier.v6
```

### 2. 美的设备配置

```yaml
# configuration.yaml

# 2.1 美的空调（通过 MQTT）
climate:
  - platform: mqtt
    name: "客厅空调"
    mode_command_topic: "midea/ac/living_room/mode/set"
    temperature_command_topic: "midea/ac/living_room/temperature/set"
    mode_state_topic: "midea/ac/living_room/mode"
    temperature_state_topic: "midea/ac/living_room/temperature"
    modes:
      - "off"
      - "cool"
      - "heat"
      - "dry"
      - "fan_only"
    min_temp: 16
    max_temp: 30
```

### 3. 格力空调配置

```yaml
# configuration.yaml

# 3.1 格力空调（通过 IR 控制）
switch:
  - platform: broadlink
    host: 192.168.1.x
    mac: 'XX:XX:XX:XX:XX:XX'
    
climate:
  - platform: smartir
    name: 卧室空调
    unique_id: bedroom_ac
    device_code: 1000
    controller_data: remote.broadlink_remote
    temperature_sensor: sensor.bedroom_temperature
    humidity_sensor: sensor.bedroom_humidity
```

### 4. 海尔设备配置

```yaml
# configuration.yaml

# 4.1 海尔设备（通过 MQTT）
climate:
  - platform: mqtt
    name: "海尔空调"
    mode_command_topic: "haier/ac/mode/set"
    temperature_command_topic: "haier/ac/temperature/set"
    mode_state_topic: "haier/ac/mode"
    temperature_state_topic: "haier/ac/temperature"
```

## 二、仪表盘设计

### 1. 仪表盘配置

```yaml
# ui-lovelace.yaml

title: 我的智能家居
views:
  # 1. 首页概览
  - title: 首页
    path: home
    badges: []
    cards:
      # 天气卡片
      - type: weather-forecast
        entity: weather.home
        
      # 快速控制
      - type: entities
        title: 快速控制
        entities:
          - entity: switch.all_lights
            name: 所有灯光
          - entity: climate.living_room
            name: 客厅空调
          - entity: climate.bedroom
            name: 卧室空调
            
      # 房间状态
      - type: glance
        title: 房间状态
        entities:
          - entity: sensor.living_room_temperature
            name: 客厅温度
          - entity: sensor.living_room_humidity
            name: 客厅湿度
          - entity: sensor.bedroom_temperature
            name: 卧室温度
          - entity: sensor.bedroom_humidity
            name: 卧室湿度

  # 2. 客厅页面
  - title: 客厅
    path: living-room
    cards:
      # 空调控制
      - type: thermostat
        entity: climate.living_room
        
      # 空气质量
      - type: gauge
        name: 空气质量
        entity: sensor.living_room_purifier_aqi
        
      # 设备状态
      - type: entities
        title: 设备状态
        entities:
          - entity: climate.living_room
          - entity: fan.living_room_purifier
          - entity: light.living_room
          
      # 温湿度历史
      - type: history-graph
        title: 温湿度变化
        entities:
          - entity: sensor.living_room_temperature
          - entity: sensor.living_room_humidity

  # 3. 卧室页面
  - title: 卧室
    path: bedroom
    cards:
      # 空调控制
      - type: thermostat
        entity: climate.bedroom
        
      # 设备状态
      - type: entities
        title: 设备状态
        entities:
          - entity: climate.bedroom
          - entity: light.bedroom
          - entity: switch.bedroom_plug
          
      # 温湿度历史
      - type: history-graph
        title: 温湿度变化
        entities:
          - entity: sensor.bedroom_temperature
          - entity: sensor.bedroom_humidity

  # 4. 自动化页面
  - title: 自动化
    path: automation
    cards:
      # 场景控制
      - type: entities
        title: 常用场景
        entities:
          - script.morning_routine
          - script.evening_routine
          - script.movie_mode
          - script.sleep_mode
          
      # 自动化状态
      - type: entities
        title: 自动化状态
        entities:
          - automation.ac_auto_control
          - automation.light_auto_control
          - automation.air_purifier_auto

### 2. 自定义主题

```yaml
# themes.yaml

modern_dark:
  # 颜色方案
  primary-color: "#4CAF50"
  accent-color: "#00897B"
  primary-background-color: "#121212"
  secondary-background-color: "#1F1F1F"
  paper-card-background-color: "#1F1F1F"
  
  # 文字颜色
  primary-text-color: "#FFFFFF"
  secondary-text-color: "#9E9E9E"
  text-primary-color: "#FFFFFF"
  
  # 卡片样式
  ha-card-border-radius: "10px"
  ha-card-box-shadow: "0px 2px 4px rgba(0, 0, 0, 0.2)"
```

### 3. 推荐的自定义卡片

```yaml
# 建议安装以下HACS前端插件：
frontend:
  extra_module_url:
    - /hacsfiles/mini-graph-card/mini-graph-card-bundle.js
    - /hacsfiles/button-card/button-card.js
    - /hacsfiles/simple-weather-card/simple-weather-card-bundle.js
    - /hacsfiles/mini-media-player/mini-media-player-bundle.js
```

## 三、自动化配置

### 1. 温度控制自动化

```yaml
# automations.yaml

# 空调智能控制
automation:
  - alias: "空调温度自动控制"
    trigger:
      - platform: numeric_state
        entity_id: sensor.living_room_temperature
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

### 2. 空气质量控制

```yaml
automation:
  - alias: "空气净化器自动控制"
    trigger:
      - platform: numeric_state
        entity_id: sensor.living_room_purifier_aqi
        above: 75
    action:
      - service: fan.turn_on
        target:
          entity_id: fan.living_room_purifier
        data:
          speed: high
```

## 四、使用建议

1. **设备命名规范**
   - 使用有意义的名称
   - 保持命名一致性
   - 按房间分类

2. **仪表盘使用技巧**
   - 最常用的控制放在首页
   - 按房间组织页面
   - 使用图表展示历史数据
   - 添加快捷场景控制

3. **性能优化**
   - 避免过多历史图表
   - 合理设置更新间隔
   - 使用缓存机制

4. **注意事项**
   - 定期备份配置
   - 记录设备 token
   - 保存设备信息 