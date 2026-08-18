# Reflection

Team chúng tôi dễ vướng nhất vào anti-pattern **"coi maintenance là tự động, không đo lại sau khi chạy"** — tin rằng `VACUUM` và `expire_snapshots` tự dọn sạch hệ thống. NB6 chứng minh ngược lại bằng số liệu đo trực tiếp trên dữ liệu tự tạo:

1. Một job crash giữa chừng để lại 3 file `.parquet` mồ côi (21.2 KB). `VACUUM` dry-run tìm thấy 211 file tombstoned, nhưng **0 trong đó** là 3 file orphan thật — `deltalake` (Rust) chỉ dọn file đã tombstone trong log, không quét thư mục vật lý như Spark `VACUUM` làm thêm. Chỉ chạy theo lịch mà không đối chiếu disk với log, rác tích luỹ vô hình.
2. `expire_snapshots` đưa Iceberg từ 20 xuống 3 snapshot, nhưng **0 file avro bị xoá** — metadata còn phình từ 324.4 KB lên 331.6 KB. Phải chạy thêm job quét stranded-manifest mới giải phóng được 36.5 KB. Bỏ bước đó, team sẽ báo cáo "đã expire mà hoá đơn S3 không giảm".

Rủi ro: không đo before/after mỗi job bảo trì, team sẽ tưởng đã dọn sạch trong khi rác âm thầm đội chi phí lưu trữ.
