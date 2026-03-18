# Báo Cáo Tiền Xử Lý Dữ Liệu - Stock Price Data Preprocessing

## Tổng Quan
Quá trình tiền xử lý được thực hiện trên 5 file CSV chứa dữ liệu giá cổ phiếu:
- **FPT.csv**
- **HPG.csv**
- **POW.csv**
- **VCB.csv**
- **VIC.csv**

---

## 1. Tải và Khám Phá Dữ Liệu (Data Loading & Exploration)
✓ Tất cả 5 file CSV đã được tải thành công
✓ Các cột chứa: time, open, close, low, high, volume
✓ Tổng số hàng cho mỗi file: 1,543 hàng dữ liệu
✓ **Missing Data Report**: Không có giá trị bị thiếu trong dữ liệu gốc

---

## 2. Xử Lý Dữ Liệu Bị Thiếu (Missing Data Handling)
**Phương pháp**: Forward Fill (ffill) và Backward Fill (bfill)
- Sử dụng để xử lý các ngày nghỉ lễ và không có giao dịch

**Kết Quả**:
| Stock | Missing Values Before | Missing Values After | Status |
|-------|----------------------|----------------------|--------|
| FPT   | 0                    | 0                    | ✓ OK   |
| HPG   | 0                    | 0                    | ✓ OK   |
| POW   | 0                    | 0                    | ✓ OK   |
| VCB   | 0                    | 0                    | ✓ OK   |
| VIC   | 0                    | 0                    | ✓ OK   |

---

## 3. Chuyển Đổi Đặc Trưng (Feature Engineering)
**Phương pháp**: Tính Daily Returns (% thay đổi giá)

**Công thức**: `daily_return = (price_today - price_yesterday) / price_yesterday`

**Lợi ích**:
- Loại bỏ tính không dừng (non-stationary) của dữ liệu giá
- Tập trung vào mức thay đổi thay vì giá tuyệt đối
- Dữ liệu stationarity - cần thiết cho các mô hình ML

**Đặc trưng được tính toán**:
1. `open_return` - % thay đổi mở cửa
2. `close_return` - % thay đổi đóng cửa
3. `low_return` - % thay đổi giá thấp
4. `high_return` - % thay đổi giá cao
5. `volume` - được giữ nguyên (không có pct_change)

**Lưu ý**: Dòng đầu tiên bị loại bỏ do NaN từ pct_change()
- Số hàng final cho mỗi stock: 1,542 hàng

---

## 4. Chia Dữ Liệu (Data Splitting)
**Tỉ lệ**: 70% Training | 20% Testing | 10% Validation

**Thứ tự chia**: Shuffle=False (chronological order - giữ thứ tự thời gian)

**Kết Quả cho mỗi Stock**:
- **Training**:   1,080 rows (70.0%)
- **Testing**:    308 rows (20.0%)
- **Validation**: 155 rows (10.0%)
- **Total**:      1,543 rows

**Lưu ý quan trọng**: Tránh Data Leakage
- Chia dữ liệu TRƯỚC khi áp dụng scaling
- Scaler được fit chỉ trên training set
- Test và validation sets được transform bằng training scaler

---

## 5. Chuẩn Hóa Dữ Liệu (Data Scaling)
**Phương pháp**: MinMaxScaler

**Công thức**: `X_scaled = (X - X_min) / (X_max - X_min)`

**Khoảng giá trị**: [0, 1]

**Quy trình**:
1. **Fit** scaler trên training data
2. **Transform** training data
3. **Transform** test data sử dụng scaler từ training
4. **Transform** validation data sử dụng scaler từ training

**Lợi ích**:
- Đưa tất cả đặc trưng về cùng khoảng giá trị
- Giúp các mô hình ML hội tụ nhanh hơn
- Tránh các đặc trưng có phạm vi lớn chiếm ưu thế

**Ví dụ dữ liệu scaled**:
```
time,open_return,close_return,low_return,high_return,volume
2020-01-03, 0.5266, 0.3779, 0.3849, 0.5284, 0.1838
2020-01-06, 0.3677, 0.4274, 0.3873, 0.3365, 0.0876
```

---

## 6. Lưu Dữ Liệu (Data Saving)
Tất cả dữ liệu đã được lưu ở:
`data/processed/scaled/[STOCK_NAME]/`

**Các file được tạo**:

### Cho mỗi Stock (5 stocks × 5 files = 25 files tổng):
```
[STOCK_NAME]_train_scaled.csv  - Training set đã scale
[STOCK_NAME]_test_scaled.csv   - Test set đã scale
[STOCK_NAME]_val_scaled.csv    - Validation set đã scale
[STOCK_NAME]_scaler.pkl        - Scaler object (để dùng cho data mới)
```

### Ví dụ cấu trúc thư mục:
```
data/processed/scaled/
├── FPT/
│   ├── FPT_train_scaled.csv
│   ├── FPT_test_scaled.csv
│   ├── FPT_val_scaled.csv
│   └── FPT_scaler.pkl
├── HPG/
│   ├── HPG_train_scaled.csv
│   ├── HPG_test_scaled.csv
│   ├── HPG_val_scaled.csv
│   └── HPG_scaler.pkl
├── POW/
├── VCB/
└── VIC/
```

---

## 7. Tóm Tắt Tiền Xử Lý
| Bước | Kết Quả | Status |
|------|---------|--------|
| Load Data | 5 files, 1,543 rows/file | ✓ Complete |
| Missing Data | No missing values → 0 rows filled | ✓ Complete |
| Daily Returns | 4 return columns + 1 volume | ✓ Complete |
| Train-Test-Val Split | 70-20-10 (1080-308-155) | ✓ Complete |
| MinMax Scaling | Range [0, 1] | ✓ Complete |
| Save Data | 25 files (5 stocks) | ✓ Complete |

---

## 8. Kích Thước Dữ Liệu
- **Training data**: 1,080 rows × 5 features/stock × 5 stocks
- **Testing data**: 308 rows × 5 features/stock × 5 stocks
- **Validation data**: 155 rows × 5 features/stock × 5 stocks

---

## 9. Lưu Ý quan trọng cho quá trình tiếp theo
1. ✓ **Data Leakage Avoidance**: Dữ liệu đã được chia trước khi scaling
2. ✓ **Scaler Objects**: Lưu các scaler để áp dụng cho dữ liệu mới/test
3. ✓ **Daily Returns**: Dữ liệu đã dừng (stationary) - phù hợp cho ML models
4. ✓ **Normalized Features**: Tất cả đặc trưng có phạm vi [0, 1]

---

## 10. Tiếp Theo
Dữ liệu đã sẵn sàng cho:
- Feature Selection
- Model Training (Linear Regression, Neural Networks, etc.)
- Time Series Forecasting
- Classification & Regression Tasks

---

**Thời gian xử lý**: Hoàn thành thành công
**Notebook**: `notebooks/02_preprocessing.ipynb`
**Data Output**: `data/processed/scaled/`
