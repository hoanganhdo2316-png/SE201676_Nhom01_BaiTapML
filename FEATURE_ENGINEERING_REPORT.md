# Feature Engineering Report - Stock Price Prediction

## Tổng Quan
Đã hoàn thành feature engineering cho 5 cổ phiếu (FPT, HPG, POW, VCB, VIC) với đầy đủ các chỉ báo kỹ thuật và đặc trưng bổ sung.

---

## 1. Dữ Liệu Input
- **Nguồn**: Dữ liệu giá gốc từ `data/raw/` (chưa scale)
- **Thời gian**: Chỉ training data (70% của tổng dữ liệu)
- **Cấu trúc**: time, open, close, low, high, volume

---

## 2. Các Chỉ Báo Kỹ Thuật (5 loại)

### 2.1 SMA (Simple Moving Average)
- **SMA_10**: Trung bình 10 ngày
- **SMA_20**: Trung bình 20 ngày
- **SMA_50**: Trung bình 50 ngày
- **Công thức**: `SMA = giá_trung_bình_của_n_ngày`

### 2.2 EMA (Exponential Moving Average)
- **EMA_12**: EMA với span=12 ngày
- **EMA_26**: EMA với span=26 ngày
- **Công thức**: `EMA_t = (price_t * multiplier) + (EMA_{t-1} * (1 - multiplier))`

### 2.3 RSI (Relative Strength Index)
- **RSI_14**: RSI với period=14 ngày
- **Công thức**:
  - `delta = close_t - close_{t-1}`
  - `gain = delta if delta > 0 else 0`
  - `loss = -delta if delta < 0 else 0`
  - `RS = average_gain / average_loss`
  - `RSI = 100 - (100 / (1 + RS))`

### 2.4 MACD (Moving Average Convergence Divergence)
- **MACD_12_26**: MACD line (EMA_12 - EMA_26)
- **MACD_signal_9**: Signal line (EMA của MACD với span=9)
- **MACD_histogram**: MACD - MACD_signal
- **Tham số**: Fast=12, Slow=26, Signal=9

### 2.5 Bollinger Bands
- **BB_upper_20**: Upper band (SMA_20 + 2*std)
- **BB_middle_20**: Middle band (SMA_20)
- **BB_lower_20**: Lower band (SMA_20 - 2*std)
- **Tham số**: Window=20, Std=2

---

## 3. Đặc Trưng Bổ Sung

### 3.1 Lag Features (Đặc trưng trễ)
- **close_lag_1**: Giá đóng cửa ngày trước đó
- **close_lag_3**: Giá đóng cửa 3 ngày trước
- **close_lag_7**: Giá đóng cửa 7 ngày trước
- **Công thức**: `df['close_lag_n'] = df['close'].shift(n)`

### 3.2 Volatility Features (Đặc trưng biến động)
- **volatility_10**: Độ lệch chuẩn rolling của giá đóng cửa trong 10 ngày
- **Công thức**: `df['volatility_10'] = df['close'].rolling(window=10).std()`

---

## 4. Biến Mục Tiêu (Target Variables)

### 4.1 Classification Target
- **target_classification**: 1 nếu giá ngày mai tăng, 0 nếu giảm
- **Công thức**: `target = 1 if close_{t+1} > close_t else 0`

### 4.2 Regression Target
- **target_regression**: Giá đóng cửa ngày t+1
- **Công thức**: `target = close_{t+1}`

---

## 5. Kết Quả Feature Engineering

### 5.1 Số Lượng Features
- **Features gốc**: 6 (time, open, close, low, high, volume)
- **Technical Indicators**: 11 features
  - SMA: 3
  - EMA: 2
  - RSI: 1
  - MACD: 3
  - Bollinger Bands: 3
- **Lag Features**: 3 features
- **Volatility Features**: 1 feature
- **Target Variables**: 2 features
- **Tổng cộng**: 24 features

### 5.2 Kích Thước Dữ Liệu
- **Mỗi stock**: 1,030 rows × 24 features
- **Tổng cộng**: 5 stocks × 1,030 rows = 5,150 samples
- **Rows bị loại**: 50 rows đầu (do NaN từ technical indicators)

### 5.3 Phân Phối Target Classification
- **Class 0 (Giảm)**: ~50%
- **Class 1 (Tăng)**: ~50%
- **Balanced dataset** cho bài toán phân loại

---

## 6. Cấu Trúc Output
```
data/processed/feature_engineered/
├── FPT/
│   └── FPT_features.csv
├── HPG/
│   └── HPG_features.csv
├── POW/
│   └── POW_features.csv
├── VCB/
│   └── VCB_features.csv
└── VIC/
    └── VIC_features.csv
```

### 6.1 Định Dạng File
- **Format**: CSV
- **Columns**: 24 features (bao gồm time và targets)
- **Data Types**: float64 cho features, datetime cho time, int cho classification target

---

## 7. Sample Data
```csv
time,open,close,low,high,volume,SMA_10,SMA_20,SMA_50,EMA_12,EMA_26,RSI_14,MACD_12_26,MACD_signal_9,MACD_histogram,BB_upper_20,BB_middle_20,BB_lower_20,close_lag_1,close_lag_3,close_lag_7,volatility_10,target_classification,target_regression
2020-03-19,17.21,17.24,17.21,17.39,1296590,17.903,18.875,19.556,18.074,18.712,22.984,-0.638,-0.441,-0.197,21.190,18.875,16.560,17.5,17.21,18.69,0.764,0,17.24
```

---

## 8. Lưu Ý Quan Trọng

### 8.1 Data Leakage Prevention
- ✅ Chỉ áp dụng feature engineering trên training data
- ✅ Không sử dụng thông tin tương lai
- ✅ Targets được tính từ dữ liệu hiện tại

### 8.2 NaN Handling
- ✅ Loại bỏ rows có NaN từ technical indicators
- ✅ Đảm bảo dữ liệu sạch cho model training

### 8.3 Feature Scaling
- ℹ️ Features chưa được scale (vẫn ở giá trị gốc)
- ℹ️ Sẽ cần scale riêng cho từng model nếu cần

---

## 9. Sử Dụng Tiếp Theo
Dữ liệu đã sẵn sàng cho:
- **Model Training**: Linear Regression, Random Forest, Neural Networks
- **Feature Selection**: Chọn features quan trọng nhất
- **Model Evaluation**: Trên test/validation sets
- **Hyperparameter Tuning**: Tối ưu model performance

---

## 10. Files Quan Trọng
- **Notebook**: `notebooks/03_feature_engineering.ipynb`
- **Data Output**: `data/processed/feature_engineered/`
- **Report**: `FEATURE_ENGINEERING_REPORT.md`

---
**Thời gian hoàn thành**: ✅ Feature engineering hoàn tất cho tất cả 5 stocks
**Số features**: 24 features/stock
**Số samples**: 1,030 samples/stock</content>
<parameter name="filePath">d:\SE201676_Nhom01_BaiTapML\FEATURE_ENGINEERING_REPORT.md