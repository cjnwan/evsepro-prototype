# 功能清单

EVSEPro 是一款新能源充电桩配套 App: 通过 BLE 直连或云端 Remote 两条通道, 对充电桩做连接、充电控制、参数配置、记录管理。本文按用户使用视角大体罗列功能, 不展开实现 (实现见 [架构文档](README.md))。

---

## 1. 顶层导航 (4 个 Tab)

[EVSRootBuilder.m](EVSEPro/App/Coordinator/EVSRootBuilder.m) 组装:

| Tab | 入口 VC | 职责 |
|---|---|---|
| 首页 Home | EVSHomeViewController | 设备连接 + 实时充电状态 + 充电控制 |
| 充电记录 Record | EVSRecordViewController | 历史记录、统计、汇总 |
| 消息 Message | EVSMessageViewController | 系统/业务消息 |
| 我的 Mine | EVSMineViewController | 账户、设置、帮助、反馈 |

---

## 2. 账户与认证

| 功能 | 入口 |
|---|---|
| 登录 | [EVSLoginViewController](EVSEPro/Module/Login/Login/EVSLoginViewController.m) |
| 注册 | [EVSRegisterViewController](EVSEPro/Module/Login/Register/EVSRegisterViewController.m) |
| 忘记密码 / 重置 | [EVSForgetPasswordViewController](EVSEPro/Module/Login/ForgetPassword/EVSForgetPasswordViewController.m) |
| 登录态保持 / 自动登录 | EVSUserManager (登录后才能用 Remote 通道、记录上传) |

> 设计细节见 `docs/superpowers/specs/2026-04-09-login-auth-design.md`。

---

## 3. 设备连接

支持 **BLE 直连** 与 **云端 Remote** 双通道, 用户可手动切换, 也会在 BLE 断开时自动降级。

| 功能 | 说明 | 入口 |
|---|---|---|
| BLE 设备扫描 / 选择 | 扫描周边充电桩, 名字过滤, 列表选择 | [EVSBluetoothDeviceListViewController](EVSEPro/Module/Home/Bluetooth/Controller/EVSBluetoothDeviceListViewController.m) |
| WiFi 设备列表 | 已配网设备走云端 Remote 控制 | [EVSWiFiDeviceListViewController](EVSEPro/Module/Home/WiFi/EVSWiFiDeviceListViewController.m) |
| 通信密码校验 | 连接需通信密码 (0x0155 错误处理) | [EVSChargerPasswordViewController](EVSEPro/Module/Charger/Setting/Controller/EVSChargerPasswordViewController.m) |
| 自动重连 | 启动 / 桩 reboot / 配网后 / 手动切换 4 场景 | 详见 [reconnect.md](reconnect.md) |
| BLE↔Remote 切换 | 手动切 + BLE 断开自动降级 | 详见 [ble-state-machine.md](ble-state-machine.md) |

---

## 4. 充电控制 (首页)

[EVSHomeViewController](EVSEPro/Module/Home/Main/Controller/EVSHomeViewController.m) + Charge 子模块。

### 4.1 实时状态展示

| 内容 | 来源命令 |
|---|---|
| 充电桩状态 (空闲/充电中/故障等) | 0x0004 StatusReport |
| 实时监控数据 (电压/电流/功率/温度) | 0x0005 MonitorData |
| 分相统计 / 仪表盘 / 充电时长 | Home/Main/View (Dashboard/PhaseStats/Status) |
| 联网状态 | 0x0127 / 0x0190 |

### 4.2 充电模式

| 模式 | 说明 | 入口 |
|---|---|---|
| 即时充电 (启停) | 一键开始 / 停止 | 0x6007 / 0x6008, 首页主按钮 |
| 单次充电 | 设定单次目标 (电量/时长) | [EVSSingleChargingViewController](EVSEPro/Module/Home/Charge/Controller/EVSSingleChargingViewController.m) |
| 自定义充电 | 自定义参数充电 | [EVSCustomChargingViewController](EVSEPro/Module/Home/Charge/Controller/EVSCustomChargingViewController.m) |
| 定时 / 重复充电 | 周期排程 (按星期 + 时间段) | [EVSRepeatedChargingViewController](EVSEPro/Module/Home/Charge/Controller/EVSRepeatedChargingViewController.m) + Schedule/TimePicker 编辑器 |
| 充电计划下发 | 计划帧下发到桩 | 0x6150 / 0x6180 ChargingPlan |
| 目标输出电流设置 | 滑杆调节充电电流上限 | 0x6107 SetGetCurrent, 首页 CurrentSetView |

---

## 5. 充电桩参数设置

Charger 模块, 多数命令 BLE / Remote 双通道可达。

| 功能 | 命令 | 入口 |
|---|---|---|
| WiFi 配网 (SSID/密码/服务器) | 0x610A | [EVSChargerWiFiViewController](EVSEPro/Module/Charger/Setting/Controller/EVSChargerWiFiViewController.m) |
| 修改通信密码 | 0x6102 | [EVSChargerPasswordViewController](EVSEPro/Module/Charger/Setting/Controller/EVSChargerPasswordViewController.m) |
| 充电桩改名 (蓝牙名) | 0x6108 | [EVSGeneralSettingsViewController](EVSEPro/Module/Charger/Setting/Controller/EVSGeneralSettingsViewController.m) |
| 电价配置 (分时电价) | — | [EVSElectricityPriceListViewController](EVSEPro/Module/Charger/Setting/Controller/EVSElectricityPriceListViewController.m) / Editor |
| 通用设置 | — | EVSGeneralSettingsViewController |
| 温度单位 (℃/℉) | 0x6112 | 设置项 |
| 屏幕亮度 | 0x6123 | 设置项 |
| HMI 屏日夜主题 | 0x6125 | 设置项 |
| 启动方式 (即插即充/刷卡等) | 0x6140 / 0x6141 | 设置项 |
| 时间 / 时区同步 | 0x6101 / 0x6170 | 自动 + 设置项 |
| 清除充电记录 | 0x6124 | 设置项 |
| 关于充电桩 (型号/序列号) | 0x6106 GetVersion | [EVSAboutChargerViewController](EVSEPro/Module/Charger/About/Controller/EVSAboutChargerViewController.m) |
| 固件 OTA 升级 | — | [EVSFirmwareUpdateViewController](EVSEPro/Module/Charger/Firmware/Controller/EVSFirmwareUpdateViewController.m) |

---

## 6. DLB 动态负载均衡

仅 DLB 型号桩, 配合光伏/电网/家庭负荷做动态分配。Charger/DLB 模块。

| 功能 | 命令 | 入口 |
|---|---|---|
| DLB 模式选择 | 0x6184 | [EVSDLBModePickerViewController](EVSEPro/Module/Charger/DLB/Controller/EVSDLBModePickerViewController.m) |
| DLB 总览 (实时分配) | 周期上报 | [EVSDLBOverviewViewController](EVSEPro/Module/Charger/DLB/Controller/EVSDLBOverviewViewController.m) |
| DLB 参数配置 | 0x6182 查询 | [EVSDLBParamsViewController](EVSEPro/Module/Charger/DLB/Controller/EVSDLBParamsViewController.m) |
| 入户总开关限流 | 0x6181 | DLB 参数页 |
| PV 额定容量设置 | 0x6183 | DLB 参数页 |
| 最大电流设置 | 0x6185 | DLB 参数页 |
| PV / 电网 / 家庭负荷实时监控 | 0x000C / 0x000D / 0x000E / 0x000F | DLB 总览页 |

> 协议见 `dlb_ble_protocol` 规格 (SDK 头文件注释引用)。

---

## 7. 充电记录

Record 模块, 详见 [record-sync.md](record-sync.md)。

| 功能 | 说明 |
|---|---|
| 记录列表 | 按 deviceId 分桶, 倒序 |
| 多通道同步 | BLE 充电结束推送 (0x0009) / 手动同步 (0x6122) / 服务端拉取 |
| 记录上传 | 指数退避重试上传到服务端 |
| 统计 / 汇总 | 能量、时长、费用统计 (Statistics / Summary 视图) |
| 时间范围筛选 | [EVSRecordRangeViewController](EVSEPro/Module/Record/Controller/EVSRecordRangeViewController.m) |
| 记录删除 / 分享 | 列表内操作 |

---

## 8. 消息中心

| 功能 | 入口 |
|---|---|
| 消息列表 (系统/业务通知) | [EVSMessageViewController](EVSEPro/Module/Message/Controller/EVSMessageViewController.m) |

---

## 9. 我的 (Mine)

| 功能 | 入口 |
|---|---|
| 个人信息 | [EVSPersonalInfoViewController](EVSEPro/Module/Mine/UserInfo/Controller/EVSPersonalInfoViewController.m) |
| 昵称修改 | [EVSNicknameViewController](EVSEPro/Module/Mine/UserInfo/Controller/EVSNicknameViewController.m) |
| 关于 App | [EVSAboutViewController](EVSEPro/Module/Mine/About/Controller/EVSAboutViewController.m) |
| FAQ 常见问题 | [EVSFAQViewController](EVSEPro/Module/Mine/FAQ/Controller/EVSFAQViewController.m) |
| 意见反馈 (含聊天) | [EVSFeedbackViewController](EVSEPro/Module/Mine/Feedback/Controller/EVSFeedbackViewController.m) / FeedbackChat |
| 隐私政策 | [EVSPrivacyViewController](EVSEPro/Module/Mine/Privacy/Controller/EVSPrivacyViewController.m) |
| 设置 (主题 浅色/深色/跟随系统) | [EVSSettingsViewController](EVSEPro/Module/Setting/Controller/EVSSettingsViewController.m), 详见 [theme.md](theme.md) |
| 导出日志 [DEBUG] | Mine 入口, 详见 [dev-tools.md](dev-tools.md) |

---

## 10. DEBUG 专用工具

仅 Debug build 编译, Release 不含。详见 [dev-tools.md](dev-tools.md)。

| 工具 | 入口 |
|---|---|
| BLE State Inspector | [EVSBLEStateInspectorViewController](EVSEPro/Module/Mine/Debug/Controller/EVSBLEStateInspectorViewController.m) |
| BLE Command Panel | [EVSBLECommandPanelViewController](EVSEPro/Module/Mine/Debug/Controller/EVSBLECommandPanelViewController.m) |

---

## 11. 控制通道总览

所有"对桩操作"的功能都有两条可能路径:

```
用户操作
   │
   ├── BLE 直连 ──→ EVSChargerCommandDispatcher ──→ 0x6xxx 命令帧 ──→ 桩
   │
   └── 云端 Remote ──→ EVSRemoteCommandService ──→ HTTP API ──→ 服务端 ──→ 桩
```

`controlMode` 决定走哪条, 自动降级与切换逻辑见 [ble-state-machine.md](ble-state-machine.md) 与 [reconnect.md](reconnect.md)。部分功能 (如 BLE 配网、固件升级) 仅 BLE 通道可用。

---

## 一句话总结

- **4 Tab**: 首页(充电控制) / 记录 / 消息 / 我的。
- **核心能力**: 设备连接(BLE+Remote 双通道) → 充电控制(即时/单次/自定义/定时/计划) → 桩参数配置(WiFi/密码/电价/DLB/固件等) → 记录管理。
- **特色**: DLB 动态负载均衡(光伏/电网/家庭负荷)、双通道自动降级、多场景自动重连。
- 实现细节见 [docs 索引](README.md) 的 4 篇架构文档。
