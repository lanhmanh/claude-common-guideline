# Checklist trước khi public app

Cập nhật lần cuối: 2026-11-14

## Bảo mật
- [ ] Không có secret trong git ([secrets-management.md](secrets-management.md))
- [ ] Bật RLS (Supabase) / Security Rules (Firebase) cho mọi bảng/collection
- [ ] HTTPS toàn bộ, SSL mode Full (strict)
- [ ] CORS chỉ cho phép domain của app
- [ ] OAuth redirect URI chỉ gồm domain thật, bỏ URI thử nghiệm không dùng
- [ ] Giới hạn tần suất (rate limit) cho API đăng nhập/gửi mail
- [ ] 2FA cho GitHub, Cloudflare, Google Cloud, Supabase/Firebase, registrar

- [ ] Đã quét bảo mật chạy tay trước release ([security-scanning.md](../05-security/security-scanning.md)) và rà [security-basics.md](../05-security/security-basics.md)

## Vận hành
- [ ] Sao lưu DB (Supabase: kiểm tra chính sách backup của gói đang dùng; tự `pg_dump` nếu cần)
- [ ] Đặt cảnh báo chi phí (Google Cloud budget alert, nhất là khi dùng Firebase Blaze)
- [ ] Biết giới hạn gói free và dự án free có bị pause khi không hoạt động không
- [ ] Theo dõi lỗi (Sentry có gói free, Firebase Crashlytics cho mobile)
- [ ] Có cách rollback: giữ bản deploy cũ, migration có kế hoạch quay lại

## Pháp lý và nội dung
- [ ] Chính sách quyền riêng tư và điều khoản sử dụng
- [ ] Nếu thu dữ liệu cá nhân người dùng Việt Nam: tìm hiểu quy định bảo vệ dữ liệu cá nhân hiện hành
- [ ] Nếu bán hàng/dịch vụ trực tuyến: thủ tục website thương mại điện tử, thuế (hỏi chuyên gia)

## Hiệu năng cơ bản
- [ ] Bật cache/CDN cho file tĩnh (Cloudflare)
- [ ] Ảnh được nén, có kích thước hợp lý
- [ ] Chạy thử trên mạng chậm và thiết bị yếu

## Phát hành
- [ ] Domain trỏ đúng, www và non-www cùng hoạt động ([domain-vn.md](domain-vn.md))
- [ ] Mobile: đủ yêu cầu store ([mobile-release.md](mobile-release.md))
- [ ] Deploy chạy tay, không bật CI tự động trên repo
