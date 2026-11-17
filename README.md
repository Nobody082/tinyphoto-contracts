# tinyphoto-contracts

Repository nguồn chuẩn cho đặc tả sản phẩm tinyphoto, Clean Architecture, design system và wire contracts. Nội dung hiện có trong `docs/` và `contracts/`; repo không chứa ứng dụng chạy độc lập.

## Chuẩn bị môi trường

Không cần framework runtime. Dùng editor có hỗ trợ Markdown, YAML và JSON. Khi sửa contract, đọc `docs/01-master-plan.md`, `docs/02-clean-architecture-guide.md`, `docs/03-design-system.md` cùng các contract liên quan trước; thay đổi wire behavior cần tương thích rõ ràng và phiên bản hóa.

## Chạy local

Không có server hay lệnh khởi chạy cho repo này. Mở tài liệu trực tiếp để xem và chỉnh sửa. Workspace hiện chưa có semantic/schema validation gate cho contracts; storage scanner không thay thế kiểm tra schema. Trước khi phát hành, cần xác thực YAML/JSON và fixtures bằng công cụ đã được pin trong CI.

## Triển khai trên cloud

Đây là tài liệu đặc tả, không phải dịch vụ cloud. Khi repo consumer được scaffold, consumer phải pin tag contracts đã phát hành và SHA-256 từng file đã dùng trong `contracts.lock.json`. Chỉ tạo release tag khi nội dung đã được review và release thực sự được chấp thuận; baseline hiện chưa có release tag. Không có GitHub Pages hoặc dịch vụ tài liệu nào được cấu hình ở repo này.

## Vận hành và khắc phục sự cố trên cloud

Khi deployment consumer báo contract mismatch, so sánh tag và SHA-256 trong `contracts.lock.json` với nội dung của bản phát hành, rồi cập nhật nguồn contract và tái tạo client/token theo quy trình consumer. Không sửa trực tiếp generated output. Nếu thay đổi breaking, tăng protocol major và chuẩn bị giai đoạn tương thích; event schema đóng cần triển khai consumer tương thích trước khi phát payload mới. Khi release hoặc CI trên cloud thất bại, kiểm tra YAML/JSON, schema refs, fixtures, checksum và tag nguồn trước khi chạy lại pipeline.
