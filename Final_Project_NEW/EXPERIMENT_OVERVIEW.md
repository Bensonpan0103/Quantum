# Experiment Overview
# 量子輔助弱訊號偵測系統之可行性研究
**生成時間：2026-05-19 20:44:50**

---

## 一、研究目標

探索量子 Hong-Ou-Mandel（HOM）電路輔助特徵萃取是否能提升弱訊號偵測的效能，
並量化量子特徵在不同 SNR 條件下的鑑別能力與計算特性。

---

## 二、整體 Pipeline 架構

```
Block 1  訊號生成 + Hilbert 相位穩定性校準
   ↓ 繼承：dataset, signal_to_angle, PHASE_STD_MAX/MIN, N_JOBS
Block 2  量子 HOM 電路設計 + CCF 基線比較
   ↓ 繼承：quantum_hom_circuit, ccf_peaks, qc_hom_vis
Block 3  特徵萃取（古典 4 + 量子 1）+ 資料增強
   ↓ 產出：train_easy/hard.csv, test_easy/hard.csv
Block 4  隨機森林消融實驗（Classical / Quantum / Mixed）
   ↓ 產出：results 字典
Block 5  1D CNN 分類器
   ↓ 產出：cnn_results 字典
Block 6  最終比較 + 三個補充實驗
   ↓ 產出：所有圖表 + PROJECT_SUMMARY.md
Block 7  實驗 Overview（本檔案）
Block 8  帶日期的結果紀錄
```

---

## 三、訊號模型（Block 1）

### SPDC 模擬模型
模擬量子光學自發參量下轉換（Spontaneous Parametric Down-Conversion）光子對：

| 訊號 | 公式 |
|------|------|
| Signal | $s(t) = \sin(2\pi f_0 t + \phi_s(t)) + n_s(t)$ |
| Idler  | $i(t) = \sin(2\pi f_0 t + \pi + \phi_i(t)) + n_i(t)$ |

- $\phi_s(t) \sim \mathcal{N}(0, 0.3)$：signal 相位雜訊
- $\phi_i(t) \sim \mathcal{N}(0, 0.15)$：idler 相位雜訊（較小，模擬量子關聯）
- $n(t)$：AWGN，依 SNR 設定功率

### 系統參數

| 參數 | 值 | 說明 |
|------|-----|------|
| 採樣率 $f_s$ | 1000 Hz | |
| 訊號長度 | 1000 點 | 1 秒 |
| 目標頻率 $f_0$ | 50 Hz | |
| SNR 範圍 | -20 ~ 6 dB | 間隔 1 dB，共 27 點 |
| Easy 標籤切點 | -8 dB | 相對容易的偵測條件 |
| Hard 標籤切點 | -15 dB | 極弱訊號偵測條件 |
| N_JOBS | 繼承自 Block 1 設定 | CPU 核心數 |

### 量子角度編碼（Hilbert 相位穩定性版 v3）

**設計演進：**
- v1（頻域能量）：`qc_hom_vis` vs `peak_value` 相關係數 = 0.94 ❌
- v2（自相關衰減）：相關係數 = 0.98 ❌
- v3（Hilbert 相位穩定性）：相關係數 = 0.91（最低）✓

**v3 計算方式：**
```python
# 1. Hilbert 轉換取瞬時相位
analytic   = hilbert(segment)
inst_phase = unwrap(angle(analytic))
phase_std  = std(diff(inst_phase))

# 2. 動態正規化（以校準資料決定範圍）
score = (PHASE_STD_MAX - phase_std) / (PHASE_STD_MAX - PHASE_STD_MIN)
theta = (pi/2) * clip(score, 0, 1)
```

**校準結果：**
| 參數 | 值 |
|------|-----|
| PHASE_STD_MAX | 1.5940 rad（低 SNR 端）|
| PHASE_STD_MIN | 0.6656 rad（高 SNR 端）|
| 動態範圍 | 0.9284 rad |

---

## 四、量子電路設計（Block 2）

### 電路架構
```
|0⟩ — RY(θ) — H — measure
```

### 解析解（可直接驗證電路正確性）
$$P(|0\rangle) = \frac{1 + \sin\theta}{2} \equiv \text{hom\_visibility}$$

### 電路驗證結果
- 理論值與實測值誤差均 < shot noise（$1/\sqrt{N_{\text{shots}}}$）
- 驗證案例：理論 0.8155，實測 0.8135，誤差 0.0020 ≪ 0.0312

### 電路參數

| 參數 | 值 |
|------|-----|
| SHOTS | 4096 |
| WINDOW（片段長度）| 200 點（頻率解析度 = 5 Hz）|
| N_SEGS（每筆片段數）| 10 |
| 模擬器 | AerSimulator（statevector method）|

---

## 五、特徵設計（Block 3）

### 特徵欄位（共 5 個）

| 特徵 | 類型 | 物理意義 | 重要性（Easy）| 重要性（Hard）|
|------|------|---------|:---:|:---:|
| peak_value | 古典 | 正規化 CCF 峰值 | 主導 | 主導 |
| fwhm | 古典 | CCF 峰值半高寬 | 低 | 中 |
| prominence | 古典 | 峰值顯著度 | 低 | 低 |
| local_cv | 古典 | 峰值附近變異係數 | 低 | 低 |
| qc_hom_vis | **量子** | HOM 電路可見度 | **0.274** | **0.168** |

### 特徵相關係數（關鍵）
- `qc_hom_vis` vs `peak_value`：r = 0.91
- 此高相關是混合架構 AUC 增益有限的根本原因

### 資料流程（無 Data Leakage）
```
1350 筆原始樣本
    ↓ Stratified split（label_easy）
Train 945 筆  ←  Test 405 筆（固定，不動）
    ↓ 資料增強（Gaussian 擾動 3% std）
Train 5000 筆
```

---

## 六、模型設計

### 隨機森林（Block 4）
```python
RandomForestClassifier(
    n_estimators=200,
    max_depth=4,
    min_samples_leaf=3,
    class_weight='balanced',
    random_state=42,
)
```
- 5-fold Stratified CV 用於訓練集驗證
- 消融實驗：Classical only / Quantum only / Mixed (All)

### 1D CNN（Block 5）
```
Input(5, 1)
→ Conv1D(32, kernel=2, padding=same, relu)
→ BatchNormalization → Dropout(0.3)
→ Conv1D(64, kernel=2, padding=same, relu)
→ BatchNormalization
→ GlobalAveragePooling1D
→ Dense(32, relu) → Dropout(0.3)
→ Dense(1, sigmoid)
```
- Optimizer：Adam(lr=1e-3)
- Loss：Binary Crossentropy
- EarlyStopping：patience=15，monitor=val_auc
- class_weight='balanced'

---

## 七、補充實驗設計（Block 6）

### 實驗 1：計算複雜度
- 測試 N = 100, 500, 1000, 2000, 5000, 10000
- 每個 N 重複 20 次取平均
- 比較 CCF（O(N log N)）vs 量子 HOM（O(1)）

### 實驗 2：Quantum only SNR 分析
- 展示 RF Quantum only 與所有模型的逐 SNR 準確率
- 標示量子特徵在哪些 SNR 點具有獨立鑑別能力

### 實驗 3：SHOTS 穩定性分析
- 測試 shots = 64, 128, 256, 512, 1024, 2048, 4096
- SNR = -20, -15, -10, -5, 0 dB
- 每個組合重複 20 次，計算 mean 和 std
- 對比理論 shot noise：$1/\sqrt{N}$

---

## 八、主要結論

| 結論 | 數據支撐 |
|------|---------|
| 量子 HOM 具備獨立偵測能力 | Quantum only AUC = 0.884（Easy）/ 0.738（Hard）|
| 混合架構增益有限 | AUC 增益 = +0.0002（Easy）/ -0.0027（Hard）|
| 量子特徵具一定重要性 | 特徵重要性 = 0.274（Easy）/ 0.168（Hard）|
| 量子計算複雜度優勢 | O(1) vs CCF 的 O(N^0.60)，在長訊號場景具理論速度優勢 |
| 量子測量符合理論 | 實測 std 與 $1/\sqrt{N}$ 高度吻合，shots≥512 時 std<0.02 |

---

## 九、未來研究方向

1. **量子照明協議**：採用真實糾纏光子對作為輸入，利用 signal-idler 聯合量測，理論上可獲得 6 dB SNR 優勢
2. **多頻帶量子特徵**：在不同頻帶各設計一個量子電路，提升特徵多樣性
3. **量子振幅估計（QAE）**：以 MLQAE 取代直接量測，在同等精度下減少量子電路呼叫次數
4. **端到端 CNN**：改以原始時域訊號作為 CNN 輸入，結合量子特徵的多模態架構
5. **真實量子硬體驗證**：在 IBM Quantum 或 IonQ 硬體上驗證模擬結果的可移植性

---

*本文件由 Block 7 自動生成，所有數字直接來自實驗結果。*
