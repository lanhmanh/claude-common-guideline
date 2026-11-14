# Quy ước cập nhật tài liệu

Mục tiêu: tài liệu luôn đúng với dịch vụ hiện tại.

## Khi nào cập nhật
- Dịch vụ đổi giao diện, đổi giới hạn gói free, đổi tên lệnh CLI.
- Xuất hiện dịch vụ/công cụ mới đáng dùng (thêm file vào `docs/02-services/`).
- Phát hiện lệnh trong tài liệu không còn chạy.

## Cách cập nhật
1. Sửa file liên quan, cập nhật dòng `Cập nhật lần cuối: YYYY-MM-DD` ở đầu file.
2. Số liệu giới hạn gói free: ghi kèm link trang giá chính thức, không chép số "cứng" nếu không chắc.
3. Thêm file mới thì thêm vào mục lục trong `README.md`.
4. Commit nhỏ, message rõ: `docs(supabase): cập nhật hạn mức free`.

## Khuôn mẫu cho một tài liệu dịch vụ
```
# Tên dịch vụ
Cập nhật lần cuối: YYYY-MM-DD
## Dùng để làm gì / Khi nào chọn
## Gói free (link trang giá chính thức)
## Setup (Windows / macOS)
## Kết nối vào app (biến môi trường, ví dụ code)
## Lỗi thường gặp
## Tham khảo
```

## Không làm
- Không thêm GitHub Actions chạy tự động (push/pull_request/schedule). Chỉ `workflow_dispatch` khi chủ repo yêu cầu.
- Không commit secret hoặc ảnh chụp có chứa key.
