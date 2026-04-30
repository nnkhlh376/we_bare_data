# Datathon E-commerce Forecasting Pipeline

## Tổng quan
Dự án này xây dựng một pipeline dự báo doanh thu và COGS cho bài toán e-commerce theo thứ tự:

1. Clean dữ liệu raw CSV sang parquet.
2. Tạo các bảng trung gian theo ngày.
3. Hợp nhất thành bảng feature cuối cùng theo ngày.
4. Huấn luyện baseline và mô hình LightGBM.
5. Tạo file submission và phần giải thích mô hình.

Toàn bộ notebook đã được viết để tự nhận diện project root nên sau khi clone về máy khác, bạn không cần sửa đường dẫn thủ công nếu giữ nguyên cấu trúc thư mục.

## Cấu trúc thư mục

- `dataset/raw/`: dữ liệu đầu vào gốc dạng CSV.
- `dataset/interim/`: dữ liệu trung gian dạng parquet.
- `dataset/interim/daily/`: các bảng theo ngày và bảng feature cuối cùng.
- `notebooks/`: toàn bộ notebook theo từng bước của pipeline.
- `outputs/`: file dự đoán, metric, submission.
- `report/`: biểu đồ và file giải thích mô hình.

## Yêu cầu môi trường

- Python 3.10 hoặc 3.11.
- Khuyến nghị dùng môi trường ảo riêng.
- Cần cài các thư viện trong `requirements.txt`.

## Cài đặt

Trên Windows PowerShell, chạy từ thư mục gốc của project:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install -r requirements.txt
```

Nếu bạn dùng Anaconda hoặc Miniconda, có thể tạo environment riêng rồi chạy:

```powershell
pip install -r requirements.txt
```

## Thứ tự chạy notebook

Hãy chạy đúng thứ tự dưới đây:

### 1. `notebooks/00_Master_Pipeline.ipynb`
Notebook này đọc toàn bộ CSV trong `dataset/raw/`, clean dữ liệu cơ bản, chuẩn hóa cột, ép kiểu ngày tháng và lưu ra parquet trong `dataset/interim/`.

Output chính:

- `dataset/interim/customers_base.parquet`
- `dataset/interim/geography_base.parquet`
- `dataset/interim/inventory_base.parquet`
- `dataset/interim/order_items_base.parquet`
- `dataset/interim/orders_base.parquet`
- `dataset/interim/payments_base.parquet`
- `dataset/interim/products_base.parquet`
- `dataset/interim/promotions_base.parquet`
- `dataset/interim/returns_base.parquet`
- `dataset/interim/reviews_base.parquet`
- `dataset/interim/sales_base.parquet`
- `dataset/interim/shipments_base.parquet`
- `dataset/interim/web_traffic_base.parquet`

### 2. `notebooks/01_build_daily_tables.ipynb`
Notebook này lấy các file `*_base.parquet` từ `dataset/interim/` rồi tạo các bảng vận hành theo ngày trong `dataset/interim/daily/`.

Output chính:

- `dataset/interim/daily/sales_daily.parquet`
- `dataset/interim/daily/web_traffic_daily.parquet`
- `dataset/interim/daily/orders_daily.parquet`
- `dataset/interim/daily/payments_daily.parquet`
- `dataset/interim/daily/order_items_daily.parquet`
- `dataset/interim/daily/promotions_daily.parquet`
- `dataset/interim/daily/returns_daily.parquet`
- `dataset/interim/daily/reviews_daily.parquet`
- `dataset/interim/daily/shipments_daily.parquet`
- `dataset/interim/daily/inventory_monthly_summary.parquet`

### 3. `notebooks/02_build_daily_feature_table_v2_ratio_features.ipynb`
Notebook này merge toàn bộ bảng daily thành một bảng feature duy nhất theo ngày, xử lý missing values và tạo ratio features phục vụ train model.

Output chính:

- `dataset/interim/daily_feature_table.parquet`
- `dataset/interim/daily_feature_table_model_safe.parquet`
- `dataset/interim/daily/model_df.parquet`

Lưu ý: `notebooks/03_baseline_and_model_LEAKAGE_SAFE.ipynb` đang đọc `dataset/interim/daily/model_df.parquet`, nên nếu notebook 03 báo thiếu file thì cần chạy lại notebook 02 cho tới cuối.

### 4. `notebooks/03_baseline_and_model_LEAKAGE_SAFE.ipynb`
Notebook này train baseline, train mô hình chính, đánh giá walk-forward validation, và tạo submission.

Output chính:

- `outputs/baseline_metrics.csv`
- `outputs/model_holdout_metrics.csv`
- `outputs/valid_predictions.csv`
- `outputs/walk_forward_metrics.csv`
- `outputs/walk_forward_predictions.csv`
- `outputs/feature_importance.csv`
- `outputs/final_report_summary.csv`
- `outputs/submission.csv`

Notebook này ưu tiên LightGBM. Nếu LightGBM không cài được, code có fallback sang `HistGradientBoostingRegressor`, nhưng để tái tạo kết quả tốt nhất nên cài LightGBM đầy đủ.

### 5. `notebooks/04_model_explainability.ipynb`
Notebook này tạo phần giải thích mô hình, feature importance, SHAP summary và checklist leakage để phục vụ báo cáo.

Output chính:

- `report/figures/feature_importance_top20_revenue.png`
- `report/figures/shap_summary_revenue.png`
- `report/feature_importance_revenue.csv`
- `report/leakage_checklist.md`

## Quy trình chạy lại từ đầu

Nếu muốn chạy lại toàn bộ pipeline từ đầu trên một máy mới, làm theo thứ tự sau:

1. Clone project về máy.
2. Mở thư mục gốc `final/` trong VS Code hoặc Jupyter.
3. Cài môi trường bằng `pip install -r requirements.txt`.
4. Chạy notebook 00 để sinh toàn bộ file parquet nền.
5. Chạy notebook 01 để tạo bảng theo ngày.
6. Chạy notebook 02 để tạo bảng feature cuối cùng.
7. Chạy notebook 03 để train model và xuất submission.
8. Chạy notebook 04 để tạo phần giải thích và báo cáo.

## Ghi chú quan trọng

- Không nên chạy notebook theo thứ tự ngẫu nhiên. Notebook 01 phụ thuộc vào output của notebook 00, notebook 02 phụ thuộc vào notebook 01, notebook 03 phụ thuộc vào notebook 02.
- Nếu gặp lỗi `FileNotFoundError`, kiểm tra xem bạn đã mở đúng thư mục gốc project chưa và đã chạy notebook trước đó hay chưa.
- Các notebook đã được viết để tự dò project root, nên không cần sửa hard-code path giữa các máy.
- File parquet được tạo ra lại sẽ ghi đè file cũ.

## Cặp thư viện đặc biệt cần chú ý

Các thư viện sau không phải thư viện chuẩn của Python, nên cần cài trong môi trường mới:

- `pandas`: đọc, xử lý dữ liệu dạng bảng.
- `numpy`: tính toán số học.
- `pyarrow`: để `pandas` có thể đọc và ghi parquet.
- `scikit-learn`: metric, validation, fallback model.
- `lightgbm`: mô hình chính trong notebook 03 và 04.
- `matplotlib`: vẽ biểu đồ trong notebook 04.
- `shap`: giải thích mô hình bằng SHAP.
- `xgboost`: có trong notebook hợp nhất và nên cài để tránh thiếu dependency khi thử nghiệm mở rộng.

## Troubleshooting nhanh

- Nếu lỗi đọc parquet, hãy kiểm tra đã cài `pyarrow` chưa.
- Nếu lỗi `lightgbm`, hãy chạy lại `pip install -r requirements.txt` trong môi trường sạch.
- Nếu notebook 03 báo thiếu `model_df.parquet`, hãy chạy lại notebook 02 đến cuối.
- Nếu notebook báo đường dẫn lệch, mở lại project từ thư mục gốc `final/` thay vì mở riêng thư mục `notebooks/`.
