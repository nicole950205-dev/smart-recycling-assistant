# AMB82-Mini + Google AI Edge Gallery Smart Recycling Assistant

## AI 智慧垃圾分類助手：專題計畫書

**Project Proposal**

本專題預計設計一套以 AMB82-mini 為實體控制器，搭配 Google AI Edge Gallery 與 Gemma 4 E2B 作為外部 AI 推理與互動層的智慧垃圾分類系統。

系統利用 AMB82-mini 的攝影機擷取待分類物品影像，透過 Wi-Fi 將影像或必要的辨識資料傳送至 Android 手機或其他相容運算裝置，再由 AI 模型分析物品類別，最後回傳分類結果。

系統會透過顯示介面、LED 或音訊提供分類建議。當基本功能穩定後，可以進一步加入伺服馬達控制的分類機構，讓使用者將垃圾放置於指定區域後，由系統協助完成分類。

本專題採用兩層式 AI 架構：AMB82-mini 負責即時硬體控制與安全機制，Android 裝置負責較高階的 AI 分析與互動流程。

---

## 1. System Architecture

```text
                    SMART RECYCLING ASSISTANT

┌──────────────────────────────────────────────┐
│                  AMB82-Mini                  │
│                                              │
│  Camera ──> Image Capture ──> Image Request  │
│                                              │
│  IR Sensor ──> Object / Presence Detection   │
│                                              │
│             Recycling Agent API              │
│          HTTP / JSON over Wi-Fi              │
│                      │                       │
│          ┌───────────┼───────────┐           │
│          ▼           ▼           ▼           │
│       Status       LED /       Servo         │
│       Control      Audio       Control       │
│                   Feedback    (Optional)     │
└──────────────────────┬───────────────────────┘
                       │ Wi-Fi
                       │
┌──────────────────────▼───────────────────────┐
│             Android Phone / Tablet           │
│                                              │
│            Google AI Edge Gallery            │
│                                              │
│            Gemma 4 E2B / AI Model            │
│                      │                       │
│             Recycling Agent Logic            │
│                      │                       │
│           Classification / Validation        │
│                      │                       │
│          ┌───────────┼───────────┐           │
│          ▼           ▼           ▼           │
│      classify()   explain()   confirm()      │
│      get_status() display()   sort()         │
└──────────────────────────────────────────────┘
```

### System Design Principles

- **AMB82-mini as the hardware controller**：負責影像擷取、感測器讀取、LED、音訊及伺服馬達控制。
- **Android as the AI reasoning layer**：執行相容的 AI 模型，負責物品分類、結果解釋與使用者互動。
- **Semantic API**：AI 僅能呼叫定義好的功能，不可直接操作任意 GPIO、PWM 或馬達參數。
- **Safety-first classification**：分類結果不確定時，要求重新拍攝或人工確認，不直接啟動自動分流。
- **Modular implementation**：影像擷取、AI 分類、結果回傳及機構控制分開設計，方便測試與擴充。

---

## 2. Why Use a Small Hardware API?

本專題不會將完整 AI 模型或複雜 Agent 框架直接放在 AMB82-mini 上，而是使用輕量化的 HTTP/JSON API 作為控制介面。

主要原因如下：

- 降低嵌入式系統的記憶體與程式複雜度。
- 讓 AI 模型可以在資源較充足的 Android 裝置上執行。
- 方便獨立測試硬體控制與 AI 分類功能。
- 避免 AI 模型直接控制不受限制的硬體參數。
- 未來可替換不同的 AI 模型，而不必大幅修改控制器韌體。

### Proposed Semantic API

| API | 功能 | 用途 |
|---|---|---|
| `GET /status` | 讀取系統狀態 | 確認裝置連線與工作狀態 |
| `GET /capture` | 擷取物品影像 | 提供 AI 分類使用 |
| `POST /classify` | 提交分類請求或結果 | 依實際架構設計影像傳輸流程 |
| `GET /result` | 取得最近一次分類結果 | 顯示分類結果 |
| `GET /led/green` | 顯示分類提示 | 表示系統已完成某個流程 |
| `GET /play/confirm` | 播放提示音 | 提示使用者確認結果 |
| `POST /sort` | 請求執行分類動作 | 進階版本使用，須經安全檢查 |

> 以上為預計設計的 API，不代表所有端點已經實作。實際傳輸方法可依 AMB82-mini 的開發環境與網路功能調整。

### Important Design Rule

AI 不應直接指定任意伺服馬達角度、PWM 佔空比或 GPIO 狀態。

例如，不應提供讓模型自由輸入硬體參數的 API：

```text
set_servo_angle(arbitrary_angle)
write_gpio(arbitrary_pin, arbitrary_value)
```

應改成有限且經過驗證的語意操作：

```text
capture()
get_status()
show_result("plastic")
request_sort("plastic")
```

所有硬體動作都必須由韌體檢查輸入值、允許的類別、機構狀態與安全條件後才能執行。

---

## 3. Two-Level AI Design

### Fast / Reactive Layer — AMB82-Mini

AMB82-mini 負責：

- 攝影機影像擷取。
- 感測器資料讀取。
- 物品是否放入拍攝區域的偵測。
- LED 或提示音控制。
- 伺服馬達控制（進階版本）。
- 通訊錯誤處理。
- 自動分流的安全互鎖。

### Cognitive / Reasoning Layer — Android Device

Android 裝置負責：

- 執行相容的 AI 模型。
- 分析待分類物品影像。
- 產生物品類別及分類建議。
- 提供分類理由或操作說明。
- 處理低信心度結果。
- 與使用者進行文字或語音互動。
- 根據使用者確認結果決定是否提出分流請求。

Google AI Edge Gallery 的實際模型支援與整合方式，須依應用程式版本、裝置資源及可用的開發介面確認。

本專題不預設 Gemma 4 E2B 可以直接在 AMB82-mini 上執行，也不假設 AI Edge Gallery 自動提供可直接呼叫的機器人 Agent API。

---

## 4. Core Features

### Feature 1: Image Capture

使用者將物品放在指定拍攝區域後，由 AMB82-mini 擷取影像。

系統可以先採用按鈕觸發拍攝，降低連續影像傳輸所造成的通訊與運算負擔。

### Feature 2: AI Waste Classification

AI 模型負責分析影像，初期只辨識三種主要類別：

- **Plastic bottle**：塑膠瓶。
- **Paper**：紙張。
- **Metal can**：金屬罐。

若影像不清楚或物品不屬於支援類別，系統應回傳 `unknown`，而不是強迫分類。

分類結果範例：

```json
{
  "label": "plastic_bottle",
  "confidence": 0.91,
  "status": "classified"
}
```

> 此 JSON 為預計使用的資料格式示例，數值僅供說明，不代表實測結果。

### Feature 3: Classification Feedback

系統根據 AI 回傳結果，透過 LED、顯示介面或音訊提供分類建議。

例如：

- 塑膠瓶：顯示「塑膠瓶」。
- 紙張：顯示「紙類」。
- 金屬罐：顯示「金屬罐」。
- 無法辨識：提示重新拍攝。

實際垃圾處理方式應依所在地的垃圾分類規定判斷，避免僅憑物品外觀就直接決定回收方式。

### Feature 4: Confidence Validation

AI 回傳分類結果後，系統會檢查分類信心度或其他可用的可信度指標。

當結果不足以支持可靠判斷時：

- 不執行自動分流。
- 提示使用者重新拍攝。
- 必要時由使用者自行確認分類。
- 將未能辨識的案例記錄於測試資料中。

信心度門檻須透過實驗設定，且不同模型的信心分數不一定能直接比較。

### Feature 5: Automatic Sorting (Optional)

進階版本可使用伺服馬達與多格分類箱，依照確認後的分類結果將物品導向指定區域。

系統應先顯示辨識結果，並在取得有效確認後才執行分類動作。

如果通訊中斷、物品位置不正確或機構狀態異常，系統應停止動作並等待人工處理。

---

## 5. Example Interaction

```text
User:
「這個物品應該怎麼分類？」

Step 1:
AMB82-mini captures an image.

Step 2:
The image is sent to the Android device.

Step 3:
The AI model analyzes the image.

Step 4:
The AI returns a structured result.

{
  "label": "plastic_bottle",
  "confidence": 0.91,
  "status": "classified"
}

Step 5:
The system displays:
「看起來是塑膠瓶，請依當地回收規定確認。」

Step 6:
The user confirms the result.

Step 7:
Optional servo mechanism performs sorting.
```

---

## 6. Testing and Evaluation

本專題將以實際測試評估系統效能。

### Experiment 1: Classification Accuracy

準備塑膠瓶、紙張及金屬罐等測試物品，將 AI 預測結果與人工標記的實際類別比較。

分類正確率：

```text
Accuracy = 正確分類數量 / 總測試數量
```

### Experiment 2: Lighting Conditions

在不同光線條件下拍攝相同物品，分析環境亮度對分類結果的影響。

### Experiment 3: Camera Angle

改變物品與攝影機的相對角度，觀察分類結果是否受到影響。

### Experiment 4: Response Time

記錄從影像拍攝、傳輸、AI 推論到顯示分類結果的總時間。

### Experiment 5: Failure Handling

測試 Wi-Fi 中斷、影像傳輸失敗、分類結果不明確及伺服機構異常等情況，確認系統不會因錯誤結果而執行不安全的動作。

---

## 7. Safety and Privacy

- 不允許 AI 直接控制任意 GPIO 或 PWM。
- 自動分流必須經過韌體的輸入驗證與安全檢查。
- 伺服馬達應設置合理的運動範圍與執行逾時機制。
- 遇到通訊中斷或異常狀態時，不應繼續執行新的分流動作。
- 預設不保存不必要的影像資料。
- 若使用手機或外部裝置分析影像，應清楚說明資料傳輸方式。
- 確認電源、馬達及其他周邊元件的電壓與電流需求。
- 實體機構應避免夾傷手指或造成其他傷害。

---

## 8. Implementation Plan

### Phase 1: Hardware Bring-up

確認 AMB82-mini 開發環境，完成 Wi-Fi 連線與 `/status` 端點。

### Phase 2: Image Capture

完成攝影機影像擷取及影像傳輸測試。

### Phase 3: AI Integration

在 Android 裝置上執行 AI 分類流程，完成分類結果的結構化回傳。

### Phase 4: Feedback

完成分類結果顯示、LED 提示或音訊回饋。

### Phase 5: Testing

進行分類正確率、反應時間及異常處理測試。

### Phase 6: Automatic Sorting

若前述功能穩定且時間允許，再加入伺服馬達與分類箱。

---

## 9. Team Responsibilities

### Two-Person Team

- **Member A**：AMB82-mini firmware, camera capture, Wi-Fi communication, hardware integration.
- **Member B**：AI classification, Android-side interaction, testing, documentation.

### Three-Person Team

- **Member A**：AMB82-mini firmware, camera, sensors and hardware control.
- **Member B**：AI model integration, classification logic and communication protocol.
- **Member C**：Sorting mechanism, experimental evaluation, GitHub documentation and demonstration.

All team members participate in integration, debugging and final presentation.

---

## 10. Expected Results

本專題預計完成一套以 AMB82-mini 為硬體控制核心的 AI 智慧垃圾分類助手原型。

基本版本將具備影像擷取、AI 分類、分類結果回傳及互動提示功能。若開發進度允許，將進一步加入自動分流機構。

專題最終成果包括：

- 可展示的硬體原型。
- AMB82-mini 韌體與通訊介面。
- AI 影像分類流程。
- 系統架構圖與操作說明。
- 分類正確率與反應時間測試紀錄。
- GitHub 專案文件及成果展示。

---

## 11. Limitations

- AI 模型的執行效能依 Android 裝置及模型版本而異。
- AMB82-mini 與 AI Edge Gallery 之間的通訊橋接需要自行開發及驗證。
- 分類結果可能受到光線、遮擋及物品外觀影響。
- 自動分流功能需要額外的機構設計與硬體整合。
- AI 分類僅提供建議，不能完全取代實際回收規定與人工判斷。
