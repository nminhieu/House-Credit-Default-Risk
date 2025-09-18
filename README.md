# Exploratory Data Analysis (EDA)

Trong giai đoạn EDA, chúng tôi tập trung phân tích đặc điểm dữ liệu từ nhiều bảng (Bureau, Bureau Balance, Application, Credit Card Balance, …) nhằm phát hiện xu hướng, bất thường và mối quan hệ giữa đặc trưng và biến mục tiêu `TARGET` (khách hàng vỡ nợ = 1, không vỡ nợ = 0).

---

## 1. Bureau EDA

### Categorical Variables
- **CREDIT_TYPE**: Nhiều loại tín dụng hiếm → gom nhóm thành `'Rare'` (trừ *Consumer credit* và *Credit card*).  
- **CREDIT_ACTIVE**: Các giá trị `'Sold'` và `'Bad Debt'` được thay bằng `'Active'` để tập trung vào tình trạng tín dụng hiện tại.  
- **CREDIT_CURRENCY** và **SK_ID_BUREAU**: Bị loại bỏ vì dư thừa hoặc không hữu ích.

### Numerical Variables
- **YEARS_CREDIT**: Phân phối tương đồng giữa hai nhóm → ít khả năng là biến dự báo mạnh.  
- **DAYS_CREDIT_ENDDATE**: Xuất hiện ngoại lai (~115 năm) → cần xử lý trong giai đoạn tiền xử lý.

---

## 2. Bureau Balance EDA

- **STATUS**:  
  - Các giá trị chính: `C` (đóng), `X` (không rõ), `0` (không quá hạn).  
  - Các giá trị `1–5` ít xuất hiện → dữ liệu mất cân bằng.  
- **MONTHS_BALANCE**:  
  - Biểu diễn mốc thời gian so với ngày nộp hồ sơ.  
  - Hầu hết tập trung ở -1 đến -5 tháng, một số hợp đồng cũ hơn ở -50 tháng.  

---

## 3. Missing Values & Data Quality

- **AMT_APPLICATION**: Thiếu ~5%.  
- **PRODUCT_COMBINATION**: Thiếu ~6%.  
- Một số bảng khác có tới **~55% cột chứa NaN**, nhiều cột vượt 70%.  
- Các biến `DAYS_*` chứa nhiều giá trị không hợp lý (`365243`, tương ứng ~1000 năm) → cần loại bỏ/ghi đè.

---

## 4. Key Observations on Features

### Application Data
- **CNT_CHILDREN**: Trung bình <1, nhưng có giá trị ngoại lai (12 con).  
- **AMT_INCOME_TOTAL**: Ngoại lai tới 117 triệu USD.  
- **DAYS_EMPLOYED**: Ngoại lai ~1000 năm.  
- **Target**: Dataset mất cân bằng (Defaulters ~8.1%, Non-defaulters ~91.9%).

### Categorical Variables
- Một số biến quan trọng với `TARGET`:  
  - `OCCUPATION_TYPE`, `ORGANIZATION_TYPE`, `NAME_INCOME_TYPE`.  
  - Khách hàng có nghề nghiệp/thu nhập ổn định thường ít vỡ nợ hơn.  
- **Địa chỉ (REGION / CITY)**: Người có địa chỉ cư trú ≠ địa chỉ làm việc/liên hệ → nguy cơ vỡ nợ cao hơn.  
- **CODE_GENDER**: Nam có tỷ lệ vỡ nợ cao hơn nữ (10.15% vs 7%).  
- **FLAG_PHONE**: Người cung cấp số điện thoại cá nhân có tỷ lệ vỡ nợ thấp hơn.  
- **WALLSMATERIAL_MODE**: Khách hàng sống trong nhà gỗ hoặc vật liệu rẻ → rủi ro vỡ nợ cao hơn.

### Numerical Variables
- **AMT_CREDIT, AMT_ANNUITY, AMT_GOODS_PRICE**: Người vỡ nợ thường vay ít hơn → gợi ý tình hình tài chính kém.  
- **DAYS_BIRTH**: Nhóm tuổi 30–40 có tỷ lệ vỡ nợ cao nhất.  
- **DAYS_REGISTRATION, DAYS_ID_PUBLISH**: Đăng ký/thay đổi giấy tờ gần ngày nộp đơn → nguy cơ vỡ nợ cao hơn.  
- **OWN_CAR_AGE**: Người mới mua xe (tuổi xe thấp) → ít vỡ nợ hơn.  
- **EXT_SOURCE_1/2/3**: Có tương quan cao với `TARGET` → biến dự báo quan trọng.

---

## 5. Conclusions from EDA
- Dataset có nhiều vấn đề chất lượng (missing values, ngoại lai, biến không hữu ích).  
- Nhiều đặc trưng phân loại mất cân bằng → cần xử lý khi feature engineering.  
- Một số đặc trưng nổi bật có giá trị dự báo tốt: `EXT_SOURCE` variables, thông tin nghề nghiệp/thu nhập, đặc điểm địa chỉ.  
- Biến thời gian (`DAYS_*`) chứa nhiều lỗi nhưng cũng tiềm ẩn tín hiệu quan trọng về rủi ro tín dụng.  
# House-Credit-Default-Risk
