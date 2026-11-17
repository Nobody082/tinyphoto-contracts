# Changelog

Chưa có release tag nào được phát hành. Các mục dưới đây mô tả nội dung workspace chưa phát hành.

## 1.1.0 (chưa phát hành)

- `openapi.yaml`, `design-tokens.json`, `01-master-plan.md`, `03-design-system.md`: giới hạn tối đa 1500px cho mỗi đơn ảnh (`SLOT` ≤ 1500×1500; tem đơn 1095×1500; photobooth 1×3 1500×3285; 2×3 3000×3285; 2×2 3000×3000; 1×4 1500×4380; giới hạn tổng thể sản phẩm tối đa 6000×6000 và ≤24M pixels cho các bố cục frame mở rộng).
- `openapi.yaml`: thêm `image/avif` vào `UploadRequest.content_type`, `UploadGrant.headers.Content-Type`
  và `Memory.content_type` (thay đổi bổ sung).
- Backend giữ reservation AVIF sau cổng vận hành `AVIF_UPLOADS_ENABLED`; khi cổng tắt trả 422 và client
  mã hóa lại trong RAM bằng PNG (tem răng cưa, có alpha) hoặc JPEG.
- `design-tokens.json`: thêm `capture.avifQuality` (0.6).
- `01-master-plan.md`, `03-design-system.md`: AVIF là định dạng ưu tiên do client mã hóa; tem răng cưa
  phải giữ alpha (AVIF có alpha hoặc PNG, không dùng JPEG). Backend không mã hóa/chuyển đổi ảnh.

Tương thích: reader cũ chỉ chấp nhận JPEG/PNG trong `Memory.content_type` sẽ từ chối AVIF. Triển khai
reader chấp nhận `image/avif` trên mọi consumer trước khi bật cổng AVIF.
