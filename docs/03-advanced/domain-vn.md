# Tên miền, gồm cả `.vn`

Cập nhật lần cuối: 2026-11-14

## Chọn đuôi tên miền
| Đuôi | Ghi chú |
|---|---|
| `.com`, `.net`, `.app`, `.dev` | Mua nhanh qua registrar quốc tế (Cloudflare Registrar, Namecheap...), không cần giấy tờ phức tạp |
| `.vn`, `.com.vn` | Do VNNIC quản lý, đăng ký qua **nhà đăng ký `.vn`** được VNNIC công bố; có xác thực danh tính chủ thể |

`.app` và `.dev` ép HTTPS (HSTS preload) nên hợp cho app web.

## Mua tên miền quốc tế
- **Cloudflare Registrar**: bán gần giá gốc, không phụ thu, DNS và HTTPS chung một nơi. Xem [cloudflare.md](../02-services/cloudflare.md).
- Kiểm tra giá năm đầu **và giá gia hạn** trước khi mua, hai mức có thể chênh lớn.

## Đăng ký `.vn`
Quy trình tổng quát (chi tiết và phí thay đổi theo quy định, hãy xác nhận với nhà đăng ký):
1. Tra tên miền còn trống trên website của một nhà đăng ký `.vn` (danh sách chính thức do VNNIC công bố tại https://www.vnnic.vn).
2. Chuẩn bị **bản khai đăng ký** và giấy tờ xác thực chủ thể. Cá nhân thường cần giấy tờ tuỳ thân; tổ chức cần giấy tờ pháp lý của đơn vị. Yêu cầu cụ thể theo Thông tư 21/2021/TT-BTTTT và văn bản thay thế/bổ sung nếu có.
3. Nộp hồ sơ cho nhà đăng ký, nhà đăng ký xác thực và chuyển VNNIC.
4. Khi được duyệt, trỏ **nameserver** về nhà cung cấp DNS bạn chọn (ví dụ Cloudflare) hoặc dùng DNS của nhà đăng ký.
5. Tra cứu thông tin chủ thể bằng WHOIS để kiểm tra.

Lưu ý:
- Lệ phí `.vn` theo biểu phí do Nhà nước ban hành (Thông tư 20/2023/TT-BTC tại thời điểm nguồn tham khảo); nhà đăng ký có thể cộng phí dịch vụ. So sánh giá giữa vài nhà đăng ký.
- Website thương mại tại Việt Nam có thể phát sinh nghĩa vụ **thông báo/đăng ký với Bộ Công Thương** (website thương mại điện tử). Hỏi tư vấn pháp lý nếu bán hàng.
- Chuyển nhà đăng ký `.vn` có thời hạn chờ sau khi đăng ký mới và trước khi hết hạn, xem quy định hiện hành.

## Trỏ tên miền về app (DNS cơ bản)
| Bản ghi | Dùng cho |
|---|---|
| `A` | Tên miền → IPv4 |
| `AAAA` | Tên miền → IPv6 |
| `CNAME` | Tên phụ → tên miền khác (vd `www` → `myapp.pages.dev`) |
| `TXT` | Xác minh quyền sở hữu, SPF/DKIM cho email |
| `MX` | Máy chủ nhận email |

Kiểm tra:
```bash
nslookup app.example.com         # Windows + macOS
dig app.example.com +short       # macOS
```
DNS cần thời gian lan truyền (từ vài phút đến vài giờ).

## Email theo tên miền
Cloudflare Email Routing chuyển tiếp `hello@domain` về Gmail miễn phí. Gửi thư từ tên miền cần cấu hình **SPF, DKIM, DMARC** (qua dịch vụ gửi mail như Resend, Brevo... xem gói free hiện hành của từng hãng).

## Checklist
- [ ] Bật tự động gia hạn, lưu email chủ thể là hộp thư bạn kiểm soát
- [ ] Bật khoá chuyển nhượng (transfer lock) và 2FA cho tài khoản registrar
- [ ] HTTPS hoạt động, SSL mode đúng ([cloudflare.md](../02-services/cloudflare.md))
