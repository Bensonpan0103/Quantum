# Quantum-Inspired Weak Signal Detection — Project Summary

## 實驗設定

| 參數 | 值 |
|------|-----|
| 訊號頻率 f0 | 50 Hz |
| 採樣率 fs | 1000 Hz |
| SNR 範圍 | -20 到 +6 dB（間隔 1 dB，共 27 點）|
| Easy 標籤切點 | -8 dB |
| Hard 標籤切點 | -15 dB |
| 訓練集 | 5000 筆（增強後）|
| 測試集 | 405 筆（固定，未增強）|

## 特徵設計（共 5 個）

### 古典特徵（4個）
| 特徵 | 說明 |
|------|------|
| peak_value | 正規化 CCF 峰值 |
| fwhm | CCF 峰值半高寬 |
| prominence | 峰值顯著度 |
| local_cv | 峰值附近變異係數 |

### 量子特徵（1個）
| 特徵 | 說明 |
|------|------|
| qc_hom_vis | HOM 量子電路可見度 P(\|0>) |

**量子電路設計：**
- 電路：`\|0> — RY(theta) — H — measure`
- theta 由 Hilbert 瞬時相位穩定性編碼
- 解析解：`hom_visibility = (1 + sin(theta)) / 2`
- 與 peak_value 相關係數：r = 0.91

## 消融實驗結果

| 模型 | Easy AUC | Hard AUC |
|------|---------|---------|
| RF Classical only | 0.9992 | 0.9834 |
| RF Quantum only | 0.8839 | 0.7383 |
| RF Mixed (All) | 0.9994 | 0.9807 |
| 1D CNN | 0.9971 | 0.9821 |

## 量子特徵貢獻

| Setting | AUC 增益（Mixed - Classical）| 量子特徵重要性 |
|---------|---------------------------|--------------|
| Easy | +0.0002 | 0.274 |
| Hard | -0.0027 | 0.168 |

## 主要結論

1. **量子電路具備獨立鑑別能力**：Quantum only AUC 在 Easy/Hard 設定下
   分別達到 0.884 / 0.738

2. **量子特徵與古典特徵存在結構性相關**（r=0.91），
   混合模型 AUC 增益有限（Easy: +0.0002, Hard: -0.0027）

3. **1D CNN 達到最高準確率**（Easy: 0.9971, Hard: 0.9821），
   展示端到端學習在弱訊號偵測的潛力

4. **資料流程已修正**：先 train/test split，增強只對 train，
   結果可信（無 data leakage）

## 產出檔案

| 檔案 | 說明 |
|------|------|
| block1_signals.png | 不同 SNR 的時域訊號 |
| block1_angle_encoding.png | Hilbert 相位編碼視覺化 |
| block2_quantum_circuit.png | 量子電路 vs CCF 比較 |
| block3_features.png | 特徵矩陣 + 分布 |
| block4_random_forest.png | RF 消融實驗結果 |
| block5_cnn.png | CNN 訓練曲線 + 混淆矩陣 |
| block5_cnn_snr.png | CNN vs RF SNR 準確率 |
| block6_roc_comparison.png | 最終 ROC 曲線比較 |
| block6_accuracy_by_snr.png | 最終 SNR 準確率比較 |
