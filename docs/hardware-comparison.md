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

本文件也直接包含台灣與淘寶採購通路、2026-09-02 新台幣價格、分階段採購順序及
進一步降價分析。目前完整台灣現貨購物車約 NT$5,300-5,600；淘寶／台灣混合採購
含集運約 NT$4,500-5,500。原本的 US$154 是設計目標，不是保證結帳價。

## 中文比較

| 功能 | Microduck 公開確認設備 | Microduck 零售參考 | DuckLite Vision v0.1 替代方案 | 數量 | DuckLite 成本 | 差異與相容性 |
|---|---|---:|---|---:|---:|---|
| 關節致動器 | 15 顆 Dynamixel XL330 系列智慧伺服；runtime 使用 XL330 control table | XL330-M288-T 每顆 $23.90；15 顆約 $358.50 | 腿部 10 顆 MG90S 金屬齒伺服，頭部及嘴部 5 顆 SG90 | 15 | $52 | **重大差異。** PWM 伺服齒隙較大，沒有原生數位回授。RP2040 必須模擬通訊，腿部需讀取內部電位器角度，行走 policy 需要重新訓練。 |
| 主電腦 | Radxa Zero 3W、RK3566、1 GB RAM、32 GB 儲存 | 1 GB、無 eMMC 版本標價 $18，目前缺貨 | Radxa Zero 3W 1 GB、無 eMMC，加 32 GB microSD | 1 | $27 | **應可相容。** 保留相同 CPU 與板卡支援；microSD 較慢也較怕突然斷電，板卡供貨是風險。 |
| 馬達及 IMU 介面 | 客製 `imu_to_dxl v2` 板、Dynamixel Protocol 2、ID 200 | 官方 PCB/BOM 未公開 | RP2040、CD74HC4067 類比多工器；PCA9685 作為可選備援 | 1 組 | $6-$10 | **需要橋接韌體。** 對主機提供原始伺服 ID 和 ID 200，1 Mbps。先驗證 RP2040 PIO 直接輸出 15 路 PWM；保留 PCA9685 接口，驗證失敗再加板。 |
| 身體姿態 | 身體 IMU 透過 ID 200 回報；量產 IMU 晶片未公開 | 未知 | MPU-6050 等級六軸 IMU，接在 RP2040 控制板 | 1 | 已含於上項 | **v0.1 應足夠。** 預期漂移和震動敏感度較高，需要校正與濾波。 |
| 第二顆／頭部 IMU | 產品規格列出兩顆 IMU；精確晶片未公開 | 未知 | 預留 I2C 接口，v0.1 不安裝 | 0 | $0 | **刪除功能。** 現階段 locomotion policy 主要使用身體 IMU；需要頭部穩定或 sensor fusion 時再加。 |
| 前置相機 | 文件唯一實機測試的是 Pi Camera v2／Sony IMX219；部分相機規格仍為 provisional | 原機採購價格未公開 | OV5647 CSI 相機模組 | 1 | $8 | **有既有 overlay 支援。** 解析度和低光品質較低，視覺模型可能需要重新調整。 |
| 深度／距離感測 | VL53L5CX 或 VL53L8CX，8x8 ToF matrix | VL53L5CX breakout 約 $22.70-$32.50 | VL53L1X 單區 ToF 模組 | 1 | $6 | **不等效。** 可維持軟體訊息格式，但沒有真正 8x8 深度圖，只適合前方障礙距離；保留 VL53L5CX 約增加 $17-$27。 |
| 音訊 | 客製 HAT 使用 TLV320AIC3104 codec；麥克風與喇叭型號未公開 | 完整模組成本未知 | INMP441 I2S 麥克風、MAX98357A 功放、2 W 喇叭 | 1 組 | $5 | **需要修改 driver/ALSA 設定。** 錄音與播放 API 可以保留，但 mixer controls 不同。 |
| NFC | 官方列出兩組 antenna；controller 和 antenna 電路未公開，目前公開 runtime 沒有 NFC driver | 未知 | 只預留接口，v0.1 不安裝 | 0 | $0 | **刪除功能。** 不影響行走、policy 部署、相機或 gamepad 操作。 |
| 電池 | 可拆式 NP-F550、2S、2600 mAh | 品牌與採購價格未公開 | 零售 NP-F550 相容電池，2200-2600 mAh | 1 | $14 | **外型與電壓相近。** 不同供應商的電芯品質、保護板與瞬間輸出能力差異很大。 |
| 電源系統 | 未公開的客製電源/HAT 電路 | 未知 | 左右腿各一組 6 V servo UBEC、獨立 5 V SBC buck、保險絲、開關及大容量電容 | 1 組 | $25-$32 | **需要實測。** 台灣可買到的低價 UBEC 常標示持續 6 A／峰值 10 A，不能把峰值當連續額定；分成左右腿兩路較合理。仍需驗證堵轉電流、雜訊和電池保護板跳脫。 |
| 配線與 carrier PCB | 客製 HAT、線束與馬達線；製造檔未公開 | 未知 | 開源 KiCad carrier、JST 類接頭、伺服線與排線 | 1 組 | $8 | **原理上可行。** 必須驗證 layout、跌倒時的接頭固定與線材應力。 |
| 機構 | 量產 CAD 未公開；只有模擬 mesh，並非完整製造 CAD | 未知 | 開源參數化 CAD、PETG/PLA 外殼、TPU 腳底與嘴部墊片、通用螺絲 | 1 組 | $12 | **全新設計。** 目標高度 21-22.5 cm、低於 500 g；MuJoCo 的質量與慣量必須重建。 |
| 控制器 | 隨附 game controller；公開軟體曾用 Xbox Wireless Controller 測試 | 隨附型號未公開 | 使用者現有 USB/Bluetooth gamepad | 1 | $0 | **應可相容。** Linux 能辨識成標準 gamepad 即可；另購控制器不計入機器人成本。 |
| **DuckLite 預估總成本** |  |  |  |  | **台灣 $167-$177；混合採購 $142-$173** | 依 2026-09-02 價格估算。混合採購已粗估 NT$200-500 集運；Radxa 缺貨溢價、分店運費與不良備品會影響結果。 |

## 採購、價格與降價空間（台灣／淘寶）

換算採 2026-09-02 的 US$1 = NT$31.728、CNY1 約 NT$4.74。拍賣平台常把最低規格、
首購補貼或 360 度伺服列為起始價，表中使用可執行的價格區間；下單時仍要重新確認
規格、庫存、運費和賣家評價。

| 功能 | 實際購買內容 | 優先通路 | 參考價格 | 下單檢查 | 降價判斷 |
|---|---|---|---:|---|---|
| 腿部伺服 | 10x MG90S、180 度、金屬齒 | 淘寶天貓電子元件企業店；急用可買[台灣現貨](https://shopee.tw/%E3%80%90%E7%92%B0%E5%B3%B6%E7%A7%91%E6%8A%80%E3%80%91-MG90S-%E8%88%B5%E6%A9%9F-%EF%BC%889g%E5%B0%BA%E5%AF%B8%EF%BC%89%E5%8E%9F%E5%BB%A0TIANKONGRC-%E5%85%A8%E9%87%91%E5%B1%AC%E9%BD%92%E8%BC%AA-14%E8%88%B5%E6%A9%9F-%E5%82%BE%E6%96%9C%E8%88%B5%E6%A9%9F-%E4%BC%BA%E6%9C%8D%E9%A6%AC%E9%81%94-i.280233910.7445670029) | 淘寶 CNY60-80／NT$284-379；台灣 NT$690-900 | 搜尋「180度、位置舵機、全金屬齒、同批次」；不要選 360 度連續旋轉版。先買 3 顆樣品。 | 淘寶可省約 NT$300-600，是最大且合理的採購價差；不能改成 SG90。 |
| 頭／嘴伺服 | 5x SG90、180 度 | 與 MG90S 淘寶同店；或 [ICShop](https://www.icshop.com.tw/categories/servo) | 淘寶 CNY15-30／NT$71-142；台灣 NT$245 | 確認 180 度位置版。 | 可省約 NT$100-170；若先做 locomotion，可延後購買。 |
| 主電腦 | Radxa Zero 3W、RK3566、1 GB | [Allnet China 官方店](https://shop.allnetchina.cn/products/copy-of-radxa-zero-3w)、Radxa 淘寶／京東官方店；台灣可看[露天搜尋](https://www.ruten.com.tw/find/?cateid=0011&q=radxa+zero+3w) | 官方正常價 US$18；1 GB + 8 GB eMMC US$22；缺貨溢價約 NT$1,100-1,400 | 必須是 Zero **3W**，不是 3E；確認 RAM、eMMC、排針與庫存。 | 正常價 eMMC 版可省 microSD，但目前供貨是最大價格風險；等待補貨優於換不相容 SBC。 |
| 儲存 | 正牌 32 GB A1 microSD | 台灣正規通路，例如 [Yahoo 購物](https://tw.buy.yahoo.com/category/38042959?flc=%E8%A8%98%E6%86%B6%E5%8D%A1&refine=ship_711&sort=price) | 約 NT$309 | 避免來源不明的假容量卡。 | 已有卡或買 eMMC 版即可省略。 |
| 即時控制 | YD-RP2040／完整 Pico 腳位相容板 | [台灣蝦皮搜尋](https://shopee.tw/search?keyword=raspberry+pi+pico) | NT$80-195 | 定版前測 USB/UART、ADC 雜訊和持續燒機。 | 台灣 NT$80 已比多數跨境零售便宜；不要用 GPIO 不足的 XIAO RP2040。 |
| PWM 產生 | 先不裝 PCA9685，但 carrier 預留接口 | 台灣現貨約 [NT$115](https://www.findprice.com.tw/b/pca9685) | NT$0；備援板 NT$115 | 先驗證 PIO 15 路同步 PWM、腳位、開機狀態與 watchdog fail-safe。 | 驗證通過可省 NT$115 和一塊板；這是待測設計選項，不是已確認結論。 |
| 角度回授 | CD74HC4067 16 路類比多工器 | 淘寶電子元件店；台灣可看[多工器搜尋](https://shopee.tw/search?keyword=%E5%A4%9A%E5%B7%A5%E5%99%A8) | 淘寶 CNY3.5-5／NT$17-24；台灣 NT$50 | 買模組而非裸 IC；需改出腿部伺服內部電位器訊號。 | 適合和其他小模組併單，單獨跨境不划算。 |
| 身體 IMU | GY-521／MPU-6050 模組 | 淘寶電子元件店或台灣材料行 | 淘寶 NT$19-38；台灣 NT$40-80 | 仿品與穩壓電路批次差異需校正。 | 價差小，依合併運費選通路。 |
| 前置相機 | OV5647 CSI 5 MP | 淘寶相機模組店；台灣可看[攝像模組搜尋](https://shopee.tw/search?keyword=%E6%94%9D%E5%83%8F%E9%A0%AD%E6%A8%A1%E5%A1%8A) | 淘寶 CNY20-30／NT$95-142；台灣 NT$172-380 | 確認 Radxa 所需 FPC 寬度、接點方向、鏡頭視角與 CSI 版本。 | 淘寶可省 NT$30-285；不要為低價誤買 USB 或夜視大板版。 |
| 距離感測 | VL53L1X 模組 | 台灣[現貨搜尋](https://shopee.tw/search?keyword=vl53l1x)或與淘寶小模組併單 | 淘寶 NT$47-85；台灣約 NT$70 | 商品頁常混列 VL53L0X；確認是 L1X 和光學蓋片。 | 台灣價已接近淘寶，到手價哪邊低就買哪邊。 |
| 音訊 | INMP441 + MAX98357A + 4 ohm 2-3 W 喇叭 | 淘寶電子元件店；台灣可看 [I2S 搜尋](https://shopee.tw/search?keyword=i2s&page=2) | 淘寶模組 NT$43-71；台灣整組約 NT$128-217 | 喇叭尺寸要配合頭部 CAD；確認 I2S 而非類比麥克風。 | 可延後到 locomotion 成立後購買。 |
| 電池 | NP-F550/F570 相容 7.2 V、約 2600 mAh | 台灣購買，[蝦皮搜尋](https://shopee.tw/search?keyword=f570) | NT$403-600 | 容量標示不代表峰值能力；實測保護板是否跳脫。 | 不建議跨境：鋰電集運限制、危險品費與退貨會吃掉價差。 |
| 伺服電源 | 2x 6 V UBEC，每顆持續 6 A／峰值 10 A | 台灣模型店，[ZTW 搜尋](https://shopee.tw/search?keyword=%2CzTw) | 約 NT$308-350／顆；兩顆 NT$616-700 | 左右腿各一顆；不能把峰值 10 A 當作連續額定。 | 不縮成一顆。這筆成本換取電壓餘量、較低線損與較容易除錯。 |
| SBC 電源 | XL4015 可調 buck，設定 5.1 V | 與淘寶小模組併單；台灣[現貨](https://shopee.tw/%2866a%29-XL4015-%E9%AB%98%E6%95%88%E7%8E%87-%E6%81%86%E5%A3%93%E6%81%86%E6%B5%815A-%E5%8F%AF%E8%AA%BF-PWM-DC-DC-%E7%A9%A9%E5%A3%93-%E9%99%8D%E5%A3%93%E5%99%A8-%E5%85%85%E9%9B%BB%E5%99%A8-%E6%A8%A1%E7%B5%84-i.581694611.12360597946) | 淘寶 NT$14-24；台灣 NT$70 | 非隔離模組；5 A 是理想上限，需散熱及 SBC 滿載壓降測試。 | 可跨境併單，但不能省略獨立 SBC rail。 |
| 保護、配線與 carrier | 保險絲、開關、低 ESR 電容、粗線、JST、首版 PCB | 接頭／螺絲可淘寶；保護件按可信規格購買 | NT$370-650 | 伺服和 SBC 分 rail、共地；保險絲放在電池後端。 | 適合合併採購，但不可用細線或不明接頭壓成本。 |
| 機構 | PLA/PETG、TPU、M2/M3 螺絲與嵌件 | 線材／緊固件淘寶，耗材依現有來源 | NT$350-500 | 未含列印機、代印費與失敗件。 | 重用耗材可省錢；不可在腿部剛性和接頭固定上過度減料。 |

跨境到手價應用「商品價 × 4.74 + 中國境內運費 + 集運 + 支付匯差 + 5-10% 備品／不良率」
估算。15 顆伺服與通用模組合箱先抓 0.8-1.5 kg、NT$200-500 集運；單項只省 NT$50
通常不值得拆成另一家店。

### 分階段採購

| 階段 | 先買什麼 | 預算 | 進入下一階段的條件 |
|---|---|---:|---|
| 1. 單腿台架 | 3x MG90S、RP2040、CD74HC4067、MPU-6050、可靠 6 V 電源、線材 | NT$700-1,100 | 完成空載／帶載追蹤誤差、齒隙、峰值電流、10 分鐘溫升及堵轉測試。 |
| 2. Locomotion 原型 | 補足腿部伺服、兩顆 UBEC、電池、保護配電與機構 | 依台架結果追加 | 雙腿 50 Hz 控制穩定，SBC rail 不因動作重開機。 |
| 3. 完整 15 DOF | 5x SG90、Radxa、儲存、相機 | 依當時 Radxa 庫存追加 | 確認 host bridge、相機 FPC 和 ONNX runtime。 |
| 4. 感知／互動 | VL53L1X、麥克風、功放與喇叭 | NT$200-300 起 | Locomotion 成立後再整合，避免非關鍵功能先吃掉預算。 |

### 為什麼考慮不用 PCA9685

| 方案 | 做法 | 優點 | 風險／代價 | v0.1 決策 |
|---|---|---|---|---|
| RP2040 + PCA9685 | RP2040 透過 I2C 命令外部 16 路 PWM 晶片 | 接線與既有函式庫簡單；RP2040 少用 15 個輸出腳位 | 多 NT$115、一塊板、I2C 更新延遲；仍需自行做失效保護 | carrier 預留接口，作為備援。 |
| RP2040 PIO 直接輸出 | 用 RP2040 的可程式化 I/O 硬體，同步產生 15 路 50 Hz servo pulse | 少一塊板、同步時序可控、降低成本 | 需要約 15 個連續 GPIO、專用韌體、watchdog 和開機低電位驗證；腳位配置很緊 | 先在單腿及 15 路假負載測試，通過才正式取消 PCA9685。 |

RP2040 的 PIO 是獨立於主 CPU 的小型可程式化 I/O 引擎，不是用一般軟體迴圈「硬湊」
15 路脈波。初步腳位預算為 15 路 servo、4 路 MUX select、1 路 ADC、2 路 IMU I2C、
2 路 host transport，共 24 個 GPIO；Pico 類板約有 26 個可用 GPIO，因此理論上可行但
餘量很小。這就是為什麼目前只能稱為「可驗證的降價選項」，不能直接視為定案。

UBEC 則不同：MG90S 啟動或堵轉會同時拉高電流。若一顆 6 A 連續型 UBEC 供應十顆腿部
伺服，可能電壓下陷、過熱或觸發保護。左右腿分兩路不是增加功能，而是在還沒有實測完整
電流波形前保留必要的電源餘量，因此目前不把縮減 UBEC 列為安全降價方案。

## English comparison

| Function | Microduck public information | Microduck retail reference | DuckLite Vision v0.1 replacement | Qty. | DuckLite target cost | Expected difference / status |
|---|---|---:|---|---:|---:|---|
| Joint actuators | 15 Dynamixel XL330-class smart servos; exact runtime uses the XL330 control table | XL330-M288-T is $23.90 each; 15 would be $358.50 | 10x MG90S metal-gear servos for legs + 5x SG90 for head and beak | 15 | $52 | **Major difference.** PWM servos have more backlash and no native digital telemetry. RP2040 must emulate the bus; leg position feedback will be added by sensing the internal potentiometers. Walking policy must be retrained. |
| Main computer | Radxa Zero 3W, RK3566, 1 GB RAM, 32 GB storage | 1 GB/no-eMMC version is listed at $18 but currently sold out | Radxa Zero 3W 1 GB/no-eMMC + 32 GB microSD | 1 | $27 | **Should work.** Same CPU family and board support. microSD is slower and less power-loss tolerant than eMMC. Availability is a sourcing risk. |
| Motor and IMU interface | Custom `imu_to_dxl v2` board, Dynamixel Protocol 2, ID 200 | Official PCB/BOM not published | RP2040 + CD74HC4067 analog multiplexer; optional PCA9685 fallback | 1 set | $6-$10 | **Firmware bridge required.** First validate direct 15-channel PWM using RP2040 PIO while keeping a PCA9685 connector/footprint as a fallback. |
| Body orientation | Body IMU behind ID 200; exact production IMU IC not public | Unknown | MPU-6050-class 6-axis module on the RP2040 board | 1 | Included above | **Likely adequate for v0.1.** More drift and vibration sensitivity are expected; calibration and filtering are required. |
| Second/head IMU | Product specification says two IMUs; exact IC not public | Unknown | Connector reserved; not populated in v0.1 | 0 | $0 | **Feature omitted.** The current locomotion policy primarily needs the body IMU. Add it if head stabilization or sensor fusion requires it. |
| Front camera | Pi Camera v2 / Sony IMX219 is the only camera documented as tested; camera details are still provisional in the press kit | Official camera pricing varies; exact bundled module price not published | OV5647 CSI camera module | 1 | $8 | **Supported with a different overlay.** Lower resolution and low-light quality, but the official setup already names the OV5647 overlay. Vision models may need retuning. |
| Depth/range sensor | VL53L5CX or VL53L8CX, 8x8 ToF output | VL53L5CX breakout: about $22.70-$32.50 | VL53L1X single-zone ToF module | 1 | $6 | **Not equivalent.** DuckLite can preserve the software message shape, but cannot recover an 8x8 depth image. Suitable for forward obstacle range only. Keeping a VL53L5CX adds roughly $17-$27. |
| Audio | TLV320AIC3104 codec on a custom HAT; microphone and speaker part numbers not public | Complete module cost unknown | INMP441 I2S microphone + MAX98357A amplifier + 2 W speaker | 1 set | $5 | **Driver/configuration change required.** Recording and playback APIs can remain the same; mixer controls will differ. |
| NFC | Two antennas are advertised; controller and antenna design are not public; no NFC driver is present in the current public runtime | Unknown | Header reserved, not populated | 0 | $0 | **Feature omitted in v0.1.** Does not block walking, policy deployment, camera, or gamepad control. |
| Battery | Removable NP-F550, nominal 2S, 2600 mAh | Brand and purchase price not published | Retail-compatible NP-F550, 2200-2600 mAh | 1 | $14 | **Mechanically and electrically similar.** Cell quality and peak-current protection vary substantially by supplier. |
| Power system | Unpublished custom power/HAT electronics | Unknown | Separate left/right 6 V servo UBECs + independent 5 V SBC buck + fuse, switch, and bulk capacitors | 1 set | $25-$32 | **Needs bench validation.** Low-cost local UBEC listings often quote 6 A continuous / 10 A peak; peak current must not be treated as a continuous rating. Split leg rails reduce risk, but stall current, noise, and battery protection trips still require measurement. |
| Wiring and carrier PCB | Custom HAT, custom harnesses, and motor cables; manufacturing files not public | Unknown | Open KiCad carrier, JST-style connectors, servo leads, and ribbon cables | 1 set | $8 | **Should work after layout verification.** Connector retention and strain relief matter during falls. |
| Structure | Unpublished production CAD; simulation meshes are available but are not manufacturing CAD | Unknown | Open parametric CAD, PETG/PLA shell, TPU soles and beak pads, commodity fasteners | 1 set | $12 | **New mechanical design.** Target is 21-22.5 cm and under 500 g so the cheaper servos remain usable. MuJoCo mass and inertia must be updated. |
| Controller | Game controller included; software has been tested with an Xbox Wireless Controller | Exact bundled model not public | Existing USB/Bluetooth gamepad | 1 | $0 | **Should work** when Linux exposes standard gamepad events. Buying a controller is optional and excluded from the robot cost. |
| **Estimated DuckLite total** |  |  |  |  | **Taiwan $167-$177; mixed-source $142-$173** | Estimate from 2026-09-02 listings. The mixed-source range includes a rough NT$200-500 freight allowance; Radxa availability and spare/failure allowance remain material. |

## 中文結論

約 $170 的台灣現貨版本可以作為實驗與開發平台，但不能先假設它和智慧伺服機器人的
性能相同。主要 cost down 來自 PWM 伺服、單區 ToF，以及暫時不安裝第二顆
IMU 和 NFC。

最大風險是腿部致動器。完整機身設計前，應先製作 hip、knee、ankle 三關節
測試架，量測 50 Hz 下的角度追蹤誤差、齒隙、峰值電流、溫度與可用扭力。
如果測試失敗，優先只升級負載最大的四個 hip-pitch 和 knee 關節，而不是直接
更換全部 15 顆伺服。

## English decision summary

The approximately $170 Taiwan-stock design is feasible as an experimental platform, but it is
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
- Taiwan USD closing rate: https://www.cbc.gov.tw/en/lp-700-2-1-40.html
- Bank of Taiwan CNY/TWD rate: https://rate.bot.com.tw/xrt?Lang=zh-TW
- Taiwan servo and module listings: https://www.icshop.com.tw/categories/servo
- Taobao MG90S price index: https://tao.hooos.com/search?w=mg90s
- Hobbywing UBEC 10A specification: https://hobbywing.oss-cn-shenzhen.aliyuncs.com/pdf/pdfcn/UBEC10A2-6S.pdf
- XL4015 application reference: https://www.xlsemi.com/demo/XL4015-DEMO.pdf

Prices and stock change. Recheck every source before ordering parts.
