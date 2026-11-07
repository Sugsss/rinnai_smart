# 林内热水器 Home Assistant 集成（Rinnai Smart / 林内智家）

> 简体中文为主文档 · [English version](README.en.md)

**中文简介**：让 Home Assistant 接入并控制林内「智家」系列燃气热水器（E86），可设定水温、运行模式、循环模式与循环预约；本仓库是上游 0.0.7 的 fork，额外修复了 Home Assistant Core 2026.11 下集成无法加载的问题。

**English summary**: Home Assistant integration for Rinnai Smart (E86) gas water heaters — temperature, operation mode and recirculation control. This fork adds a compatibility fix for Home Assistant Core 2026.11.

| 项目 | 地址 |
|---|---|
| 上游仓库 | [catro/rinnai_smart](https://github.com/catro/rinnai_smart)（v0.0.7） |
| 本仓库（可安装版） | [Sugsss/rinnai_smart](https://github.com/Sugsss/rinnai_smart) |
| 兼容修复 PR | [catro/rinnai_smart#19](https://github.com/catro/rinnai_smart/pull/19) |
| 兼容版 Release | [0.0.7-ha2026.11](https://github.com/Sugsss/rinnai_smart/releases/tag/0.0.7-ha2026.11) |

---

## ⚠️ 使用前必读

| 风险 | 说明 |
|---|---|
| 无官方 API | 林内没有提供官方开放接口，云端协议随时可能变更并导致集成失效 |
| 账号风险 | 使用风险自负，账号存在被封禁的可能 |
| Alpha 状态 | 集成仍处于 alpha；一旦连接丢失，需要重新配置设备 |
| 机型限制 | 目前仅支持 **RUS-R16E86FBF** |

## 功能

- 热水器实体：设定目标水温（°C）、切换运行模式
- 循环功能：循环模式、循环预约时间、临时 / 一键循环
- 支持同一账号下多台林内设备
- 配置流程自带简体中文界面（`custom_components/rinnai_smart/translations/zh-Hans.json`）

![](./screenshot.png)

## 安装

### 版本选择

`main` 分支属于 alpha 质量，不保证稳定。建议通过 HACS 安装正式 release，变更记录见
[本仓库 Releases](https://github.com/Sugsss/rinnai_smart/releases)；上游原始版本见
[上游 Releases](https://github.com/catro/rinnai_smart/releases)。

### HACS（推荐）

[![在 HACS 中打开本仓库](https://my.home-assistant.io/badges/hacs_repository.svg)](https://my.home-assistant.io/redirect/hacs_repository/?owner=Sugsss&repository=rinnai_smart&category=integration)

把本仓库添加为 HACS 自定义仓库（类别：**集成 / Integration**），选择 `0.0.7-ha2026.11` 或更新版本安装。

### 手动安装

1. 把仓库里的 `custom_components/rinnai_smart` 目录复制到 HA 的 `custom_components/` 下
2. 重启 Home Assistant
3. 按下面「配置」添加集成

> ⚠️ 手动安装收不到新版本提醒，需要自行订阅本仓库的 Release。

## 配置

[![在 Home Assistant 中开始配置](https://my.home-assistant.io/badges/config_flow_start.svg)](https://my.home-assistant.io/redirect/config_flow_start/?domain=rinnai)

若按钮不可用：**设置 → 设备与服务 → 右下角「添加集成」→ 搜索 `Rinnai Smart` / `林内智家`**，
用林内智家 App 的手机号和密码登录。

## 设备与实体筛选

配置流程先验证账号，再分两步筛选：

1. **设备**：按账号里的名称与 MAC 选择热水器。新配置默认「包含」且全选，之后账号新增的设备不会自动接入。
2. **实体**：按设备勾选具体功能（热水器实体、燃烧状态、循环开关等）。默认「排除」且无排除项，即全部接入。

两种模式：

- **包含（Include）**：只接入勾选项；留空 = 一个都不接入
- **排除（Exclude）**：接入勾选项以外的全部；留空 = 全部接入，且允许未来新增项

实体选择按设备保存、使用稳定 unique ID，改名不影响选择；过滤掉模式选择器不会移除热水器实体的运行模式功能。

之后可在「设置 → 设备与服务 → Rinnai Smart → 配置」重新筛选，无需再输密码。两步都保存才生效，取消则保持原状；云端暂时查不到的已保存项仍以 ID 显示。保存后集成会重载，先关闭旧 MQTT 连接再建立新连接。

旧的配置条目（没有筛选项）维持原来的「全接入」行为。未选中的设备 / 实体不再请求状态、也不订阅 MQTT，但设备发现仍需要云端设备列表；故意留空是合法配置，此时不启动 MQTT。

被排除的既有实体会**被集成禁用**，注册表记录、实体 ID、自定义名称和设备记录都会保留；重新勾选即可恢复，手动禁用的实体不会被自动启用。引用这些实体的仪表盘卡片 / 自动化需要自行调整。

## 开发测试

Python 3.14，把 `requirements_test.txt` 装进虚拟环境后运行：

```bash
python -m pytest -q --disable-socket --allow-unix-socket
```

测试使用合成设备数据与 mock 的 HTTP/MQTT，覆盖 HA 的 flow manager、平台加载、实体注册表与选项重载；不连林内云端，也不控制真实热水器。

## HA 2026.11 兼容修复（本 fork 独有）

**问题**：HA Core 2026.11 移除了 `homeassistant.components.water_heater` 对 `PRECISION_WHOLE` 的再导出。上游 0.0.7 仍从该模块导入，导致 `water_heater.py` 在导入阶段就失败，`async_setup_entry` 转发平台中断，整个配置条目报 `Error setting up entry ... for rinnai`，五个平台的实体全部不可用。

**修法**（一行）：改从 `homeassistant.const` 导入，该常量仍在（值为 `1`）；旧版 HA 也只是再导出同一个常量，因此向后兼容。

```python
# 修改前（HA 2026.11 下报错）
from homeassistant.components.water_heater import PRECISION_WHOLE
# 修改后
from homeassistant.const import PRECISION_WHOLE
```

**验证**（HAOS 2026.11.0 真机）：重启后无 `Error setting up entry`、无 ERROR 日志，`water_heater` 平台成功导入，7 个 `rinnai` 实体全部启用；容器内 9 个平台模块导入均通过。

**上游**：[PR #19](https://github.com/catro/rinnai_smart/pull/19)（另有改法相同的 [#18](https://github.com/catro/rinnai_smart/pull/18)）。任一合并后，建议回归官方 release。

### 安装这个版本的注意事项

仓库目录是 `custom_components/rinnai_smart`，而 `manifest.json` 里的 `domain` 是 `rinnai`。HA 按 manifest domain 索引自定义集成，所以已有的「林内智家」配置条目依然能匹配。

> ⚠️ **同一台机器不要同时保留 `custom_components/rinnai` 和 `custom_components/rinnai_smart`**：两者声明同一个 domain，会冲突。装新版前先删除 / 改名旧目录，或者继续沿用已经修好的 `custom_components/rinnai`。

HACS 添加本仓库为自定义「集成」仓库，安装 `0.0.7-ha2026.11` release；`main` 也是同一份修复后的代码。

### 已知待办（本次未改）

- HA 2026.11 提示 `_attr_temperature_unit`、`target_temperature` 已弃用，2027.11 起失效，需改用 `native_*`
- 平台转发时会记录一条 `blocking import_module` 警告

## 相关链接

| 内容 | 地址 |
|---|---|
| 本仓库（中文主文档） | https://github.com/Sugsss/rinnai_smart |
| 兼容版 Release | https://github.com/Sugsss/rinnai_smart/releases/tag/0.0.7-ha2026.11 |
| 上游仓库 | https://github.com/catro/rinnai_smart |
| 兼容修复 PR #19 | https://github.com/catro/rinnai_smart/pull/19 |
| 英文文档 | [README.en.md](README.en.md) |
