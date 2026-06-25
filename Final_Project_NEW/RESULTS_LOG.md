# 實驗結果日誌
# Quantum-Assisted Weak Signal Detection — Results Log

此檔案累積記錄每次實驗的完整結果。
每次執行 Block 8 都會自動 append 一筆新紀錄，不會覆蓋舊紀錄。


---

## 實驗紀錄 `20260519_204450`
**執行時間：2026-05-19 20:44:50**

### 實驗設定

| 參數 | 值 |
|------|-----|
| SNR 範圍 | -20 ~ 6 dB（共 27 點）|
| Easy cutoff | -8 dB |
| Hard cutoff | -15 dB |
| 量子電路 SHOTS | 4096 |
| 片段長度 WINDOW | 200 點 |
| 每筆片段數 N_SEGS | 10 |
| 特徵欄位 | peak_value, fwhm, prominence, local_cv, qc_hom_vis |
| N_JOBS | -1 |

### 量子角度編碼校準

| 參數 | 值 |
|------|-----|
| PHASE_STD_MAX | 1.594 rad |
| PHASE_STD_MIN | 0.6656 rad |
| 動態範圍 | 0.9284 rad |

### 消融實驗結果（Test AUC）

| 模型 | Easy CV | Easy Test | Hard CV | Hard Test |
|------|:-------:|:---------:|:-------:|:---------:|
| RF Classical only | 0.9994 | 0.9992 | 0.9916 | 0.9834 |
| RF Quantum only | 0.9246 | 0.8839 | 0.8106 | 0.7383 |
| RF Mixed (All) | 0.9992 | 0.9994 | 0.9913 | 0.9807 |
| 1D CNN | — | 0.9971 | — | 0.9821 |

### 量子特徵貢獻分析

| 指標 | Easy | Hard |
|------|:----:|:----:|
| AUC 增益（Mixed - Classical）| +0.0002 | -0.0027 |
| 量子特徵重要性（Feature Importance）| 0.2736 | 0.1682 |

### CNN 訓練資訊

| 設定 | Test AUC | 訓練 Epochs |
|------|:--------:|:-----------:|
| Easy | 0.9971 | 98 |
| Hard | 0.9821 | 78 |

### 本次實驗備註

> *(請在此手動填入本次實驗的備註，例如：修改了哪個參數、觀察到什麼現象等)*

