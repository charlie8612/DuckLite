# DuckLite Vision v0.1 硬體替代與成本比較 / Hardware replacement and cost comparison

Updated: 2026-09-02

更新日期：2026-09-02

本文件比較 Pollen Robotics 公開資料中可確認的 Microduck 硬體需求，與
DuckLite Vision v0.1 的低成本替代方案。Microduck 沒有公開完整量產 BOM，
所以未知的料號與價格會直接標示未知，不使用推測補齊。

DuckLite 成本使用單件零售、自行組裝估價，單位為美元；不含稅金、運費、工具、
3D 列印機費用及試作耗損，也不是大量採購價格。

This document compares the hardware requirements publicly visible in Pollen
Robotics' Microduck software and product documentation with the proposed
DuckLite Vision v0.1 implementation. It does not claim to reproduce Microduck's
unpublished production BOM.

Costs are estimated single-unit retail costs in USD, before tax, shipping, tools,
and failed prototype parts. A listed price is not a volume-purchase price.

## 中文比較

| 功能 | Microduck 公開確認設備 | Microduck 零售參考 | DuckLite Vision v0.1 替代方案 | 數量 | DuckLite 成本 | 差異與相容性 |
|---|---|---:|---|---:|---:|---|
| 關節致動器 | 15 顆 Dynamixel XL330 系列智慧伺服；runtime 使用 XL330 control table | XL330-M288-T 每顆 $23.90；15 顆約 $358.50 | 腿部 10 顆 MG90S 金屬齒伺服，頭部及嘴部 5 顆 SG90 | 15 | $52 | **重大差異。** PWM 伺服齒隙較大，沒有原生數位回授。RP2040 必須模擬通訊，腿部需讀取內部電位器角度，行走 policy 需要重新訓練。 |
| 主電腦 | Radxa Zero 3W、RK3566、1 GB RAM、32 GB 儲存 | 1 GB、無 eMMC 版本標價 $18，目前缺貨 | Radxa Zero 3W 1 GB、無 eMMC，加 32 GB microSD | 1 | $27 | **應可相容。** 保留相同 CPU 與板卡支援；microSD 較慢也較怕突然斷電，板卡供貨是風險。 |
| 馬達及 IMU 介面 | 客製 `imu_to_dxl v2` 板、Dynamixel Protocol 2、ID 200 | 官方 PCB/BOM 未公開 | Raspberry Pi Pico/RP2040、PCA9685、類比多工器 | 1 組 | $10 | **需要橋接韌體。** 對主機提供原始伺服 ID 和 ID 200，1 Mbps；協定相容但電路不同。 |
| 身體姿態 | 身體 IMU 透過 ID 200 回報；量產 IMU 晶片未公開 | 未知 | MPU-6050 等級六軸 IMU，接在 RP2040 控制板 | 1 | 已含於上項 | **v0.1 應足夠。** 預期漂移和震動敏感度較高，需要校正與濾波。 |
| 第二顆／頭部 IMU | 產品規格列出兩顆 IMU；精確晶片未公開 | 未知 | 預留 I2C 接口，v0.1 不安裝 | 0 | $0 | **刪除功能。** 現階段 locomotion policy 主要使用身體 IMU；需要頭部穩定或 sensor fusion 時再加。 |
| 前置相機 | 文件唯一實機測試的是 Pi Camera v2／Sony IMX219；部分相機規格仍為 provisional | 原機採購價格未公開 | OV5647 CSI 相機模組 | 1 | $8 | **有既有 overlay 支援。** 解析度和低光品質較低，視覺模型可能需要重新調整。 |
| 深度／距離感測 | VL53L5CX 或 VL53L8CX，8x8 ToF matrix | VL53L5CX breakout 約 $22.70-$32.50 | VL53L1X 單區 ToF 模組 | 1 | $6 | **不等效。** 可維持軟體訊息格式，但沒有真正 8x8 深度圖，只適合前方障礙距離；保留 VL53L5CX 約增加 $17-$27。 |
| 音訊 | 客製 HAT 使用 TLV320AIC3104 codec；麥克風與喇叭型號未公開 | 完整模組成本未知 | INMP441 I2S 麥克風、MAX98357A 功放、2 W 喇叭 | 1 組 | $5 | **需要修改 driver/ALSA 設定。** 錄音與播放 API 可以保留，但 mixer controls 不同。 |
| NFC | 官方列出兩組 antenna；controller 和 antenna 電路未公開，目前公開 runtime 沒有 NFC driver | 未知 | 只預留接口，v0.1 不安裝 | 0 | $0 | **刪除功能。** 不影響行走、policy 部署、相機或 gamepad 操作。 |
| 電池 | 可拆式 NP-F550、2S、2600 mAh | 品牌與採購價格未公開 | 零售 NP-F550 相容電池，2200-2600 mAh | 1 | $14 | **外型與電壓相近。** 不同供應商的電芯品質、保護板與瞬間輸出能力差異很大。 |
| 電源系統 | 未公開的客製電源/HAT 電路 | 未知 | 6 V 高電流 servo UBEC、獨立 5 V SBC buck、保險絲、開關及大容量電容 | 1 組 | $12 | **需要實測。** 分開供電可降低 SBC 重開機機率，但仍需驗證堵轉電流、雜訊和電池保護板跳脫。 |
| 配線與 carrier PCB | 客製 HAT、線束與馬達線；製造檔未公開 | 未知 | 開源 KiCad carrier、JST 類接頭、伺服線與排線 | 1 組 | $8 | **原理上可行。** 必須驗證 layout、跌倒時的接頭固定與線材應力。 |
| 機構 | 量產 CAD 未公開；只有模擬 mesh，並非完整製造 CAD | 未知 | 開源參數化 CAD、PETG/PLA 外殼、TPU 腳底與嘴部墊片、通用螺絲 | 1 組 | $12 | **全新設計。** 目標高度 21-22.5 cm、低於 500 g；MuJoCo 的質量與慣量必須重建。 |
| 控制器 | 隨附 game controller；公開軟體曾用 Xbox Wireless Controller 測試 | 隨附型號未公開 | 使用者現有 USB/Bluetooth gamepad | 1 | $0 | **應可相容。** Linux 能辨識成標準 gamepad 即可；另購控制器不計入機器人成本。 |
| **DuckLite 預估總成本** |  |  |  |  | **$154** | 完成實際購物車和單腿測試前，合理誤差約為 +/- $20。 |

## English comparison

| Function | Microduck public information | Microduck retail reference | DuckLite Vision v0.1 replacement | Qty. | DuckLite target cost | Expected difference / status |
|---|---|---:|---|---:|---:|---|
| Joint actuators | 15 Dynamixel XL330-class smart servos; exact runtime uses the XL330 control table | XL330-M288-T is $23.90 each; 15 would be $358.50 | 10x MG90S metal-gear servos for legs + 5x SG90 for head and beak | 15 | $52 | **Major difference.** PWM servos have more backlash and no native digital telemetry. RP2040 must emulate the bus; leg position feedback will be added by sensing the internal potentiometers. Walking policy must be retrained. |
| Main computer | Radxa Zero 3W, RK3566, 1 GB RAM, 32 GB storage | 1 GB/no-eMMC version is listed at $18 but currently sold out | Radxa Zero 3W 1 GB/no-eMMC + 32 GB microSD | 1 | $27 | **Should work.** Same CPU family and board support. microSD is slower and less power-loss tolerant than eMMC. Availability is a sourcing risk. |
| Motor and IMU interface | Custom `imu_to_dxl v2` board, Dynamixel Protocol 2, ID 200 | Official PCB/BOM not published | Raspberry Pi Pico/RP2040 + PCA9685 + analog multiplexer | 1 set | $10 | **Firmware bridge required.** It will expose the original servo IDs and ID 200 at 1 Mbps. This is protocol-compatible, not electrically identical. |
| Body orientation | Body IMU behind ID 200; exact production IMU IC not public | Unknown | MPU-6050-class 6-axis module on the RP2040 board | 1 | Included above | **Likely adequate for v0.1.** More drift and vibration sensitivity are expected; calibration and filtering are required. |
| Second/head IMU | Product specification says two IMUs; exact IC not public | Unknown | Connector reserved; not populated in v0.1 | 0 | $0 | **Feature omitted.** The current locomotion policy primarily needs the body IMU. Add it if head stabilization or sensor fusion requires it. |
| Front camera | Pi Camera v2 / Sony IMX219 is the only camera documented as tested; camera details are still provisional in the press kit | Official camera pricing varies; exact bundled module price not published | OV5647 CSI camera module | 1 | $8 | **Supported with a different overlay.** Lower resolution and low-light quality, but the official setup already names the OV5647 overlay. Vision models may need retuning. |
| Depth/range sensor | VL53L5CX or VL53L8CX, 8x8 ToF output | VL53L5CX breakout: about $22.70-$32.50 | VL53L1X single-zone ToF module | 1 | $6 | **Not equivalent.** DuckLite can preserve the software message shape, but cannot recover an 8x8 depth image. Suitable for forward obstacle range only. Keeping a VL53L5CX adds roughly $17-$27. |
| Audio | TLV320AIC3104 codec on a custom HAT; microphone and speaker part numbers not public | Complete module cost unknown | INMP441 I2S microphone + MAX98357A amplifier + 2 W speaker | 1 set | $5 | **Driver/configuration change required.** Recording and playback APIs can remain the same; mixer controls will differ. |
| NFC | Two antennas are advertised; controller and antenna design are not public; no NFC driver is present in the current public runtime | Unknown | Header reserved, not populated | 0 | $0 | **Feature omitted in v0.1.** Does not block walking, policy deployment, camera, or gamepad control. |
| Battery | Removable NP-F550, nominal 2S, 2600 mAh | Brand and purchase price not published | Retail-compatible NP-F550, 2200-2600 mAh | 1 | $14 | **Mechanically and electrically similar.** Cell quality and peak-current protection vary substantially by supplier. |
| Power system | Unpublished custom power/HAT electronics | Unknown | 6 V high-current servo UBEC + isolated 5 V SBC buck + fuse, switch, and bulk capacitors | 1 set | $12 | **Needs bench validation.** Separate rails reduce SBC resets, but servo stall current, noise, and battery protection trips remain risks. |
| Wiring and carrier PCB | Custom HAT, custom harnesses, and motor cables; manufacturing files not public | Unknown | Open KiCad carrier, JST-style connectors, servo leads, and ribbon cables | 1 set | $8 | **Should work after layout verification.** Connector retention and strain relief matter during falls. |
| Structure | Unpublished production CAD; simulation meshes are available but are not manufacturing CAD | Unknown | Open parametric CAD, PETG/PLA shell, TPU soles and beak pads, commodity fasteners | 1 set | $12 | **New mechanical design.** Target is 21-22.5 cm and under 500 g so the cheaper servos remain usable. MuJoCo mass and inertia must be updated. |
| Controller | Game controller included; software has been tested with an Xbox Wireless Controller | Exact bundled model not public | Existing USB/Bluetooth gamepad | 1 | $0 | **Should work** when Linux exposes standard gamepad events. Buying a controller is optional and excluded from the robot cost. |
| **Estimated DuckLite total** |  |  |  |  | **$154** | Target uncertainty is approximately +/- $20 until a real cart and one-leg bench are completed. |

## 中文結論

約 $150 的版本可以作為實驗與開發平台，但不能先假設它和智慧伺服機器人的
性能相同。主要 cost down 來自 PWM 伺服、單區 ToF，以及暫時不安裝第二顆
IMU 和 NFC。

最大風險是腿部致動器。完整機身設計前，應先製作 hip、knee、ankle 三關節
測試架，量測 50 Hz 下的角度追蹤誤差、齒隙、峰值電流、溫度與可用扭力。
如果測試失敗，優先只升級負載最大的四個 hip-pitch 和 knee 關節，而不是直接
更換全部 15 顆伺服。

## English decision summary

The approximately $150 design is feasible as an experimental platform, but it is
not performance-equivalent to a smart-servo robot. Its three deliberate cost cuts
are PWM servos, a single-zone ToF sensor, and omission of the second IMU and NFC.

The highest technical risk is the leg actuator system. Before designing the full
body, build one hip-knee-ankle chain and measure tracking error, backlash, peak
current, temperature, and usable torque at 50 Hz. If that bench fails, upgrade only
the four most loaded hip-pitch and knee joints rather than replacing all 15 servos.

## 價格與相容性來源 / Price and compatibility references

- Microduck software: https://github.com/pollen-robotics/microduck
- Microduck RL and simulation assets: https://github.com/pollen-robotics/microduck_rl
- Microduck press kit and open-source scope: https://pollen-robotics.com/microduck/press-kit/
- ROBOTIS XL330-M288-T: https://en.robotis.com/shop_en/item.php?it_id=902-0163-000
- Radxa Zero 3W listing: https://shop.allnetchina.cn/products/copy-of-radxa-zero-3w
- Raspberry Pi camera documentation: https://www.raspberrypi.com/documentation/accessories/camera.html
- VL53L5CX retail reference: https://www.digikey.com/en/products/detail/sparkfun-electronics/18642/15775141
- Raspberry Pi Pico: https://www.raspberrypi.com/news/raspberry-pi-silicon-pico-now-on-sale/
- MG90S retail reference: https://www.circuitspecialists.com/mg90s-digital-servo

Prices and stock change. Recheck every source before ordering parts.
