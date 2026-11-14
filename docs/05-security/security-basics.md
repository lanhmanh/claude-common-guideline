# Bảo mật cơ bản cho app (từ local đến public)

Cập nhật lần cuối: 2026-11-14

Mục tiêu: nắm các lỗi phổ biến nhất và biện pháp phòng thủ tối thiểu. Đọc cùng [security-scanning.md](security-scanning.md) (quét tự động) và [go-live-checklist.md](../03-advanced/go-live-checklist.md).

## 1. Tư duy nền tảng
- **Không tin dữ liệu từ client.** Mọi kiểm tra quyền và dữ liệu phải làm ở server (hoặc RLS/Security Rules), không chỉ ở giao diện.
- **Quyền tối thiểu (least privilege):** mỗi tài khoản, token, service chỉ có quyền cần dùng.
- **Phòng thủ nhiều lớp:** một lớp hỏng thì còn lớp khác.
- **Mặc định an toàn:** tắt mọi thứ không dùng, đóng cổng không cần.
- **Giả định secret sẽ lộ:** thiết kế để đổi key nhanh ([secrets-management.md](../03-advanced/secrets-management.md)).

## 2. OWASP Top 10:2025 (rủi ro web phổ biến)
Danh sách do OWASP công bố, cập nhật mỗi vài năm, bản 2025 là bản hiện hành. Tên chính thức từng mục: https://owasp.org/Top10/2025/

| # | Rủi ro | Phòng thủ tối thiểu |
|---|---|---|
| A01 | Broken Access Control (kiểm soát truy cập, gồm cả SSRF) | Kiểm tra quyền ở server cho **từng** request; mặc định từ chối; RLS (Supabase), Security Rules (Firebase) |
| A02 | Security Misconfiguration | Tắt debug ở production, bỏ tài khoản/mật khẩu mặc định, rà cấu hình cloud, bật header bảo mật |
| A03 | Software Supply Chain Failures | Ghim version, file lock, quét dependency, cẩn trọng package mới/ít người dùng |
| A04 | Cryptographic Failures | HTTPS mọi nơi, băm mật khẩu bằng thuật toán chuyên dụng, không tự chế mã hoá |
| A05 | Injection (SQL, lệnh, XSS) | Câu lệnh tham số hoá, escape đầu ra, không ghép chuỗi vào SQL/shell |
| A06 | Insecure Design | Nghĩ đến kịch bản lạm dụng ngay lúc thiết kế (rate limit, giới hạn, xác nhận) |
| A07 | Authentication Failures | Dùng dịch vụ auth có sẵn, 2FA, chống brute force, cookie an toàn |
| A08 | Software or Data Integrity Failures | Không chạy code/plugin không tin cậy, kiểm tra tính toàn vẹn artifact |
| A09 | Security Logging & Alerting Failures | Ghi log sự kiện bảo mật (đăng nhập, đổi quyền), có cảnh báo |
| A10 | Mishandling of Exceptional Conditions | Xử lý lỗi đúng cách, không lộ stack trace, lỗi thì mặc định "đóng" |

Nguồn: bản 2025 có hai mục mới là Software Supply Chain Failures và Mishandling of Exceptional Conditions, SSRF được gộp vào Broken Access Control, Security Misconfiguration lên hạng 2.

## 3. Xác thực và phiên đăng nhập
- **Ưu tiên dịch vụ có sẵn** (Supabase Auth, Firebase Auth, Google OAuth: [google-oauth.md](../02-services/google-oauth.md)) thay vì tự viết.
- Nếu buộc phải lưu mật khẩu: băm bằng **argon2id** (hoặc bcrypt/scrypt), có salt riêng. Không dùng MD5/SHA-1/SHA-256 trần.
- Bật **2FA/passkey** cho tài khoản quản trị và tài khoản dịch vụ (GitHub, Cloudflare, Google, registrar).
- Cookie phiên: `HttpOnly`, `Secure`, `SameSite=Lax` (hoặc `Strict`).
- JWT: kiểm tra chữ ký **và** hạn dùng, chỉ chấp nhận thuật toán định trước, hạn ngắn + refresh token. Không đặt dữ liệu nhạy cảm trong payload (JWT chỉ được mã hoá base64, không bí mật).
- Giới hạn số lần thử đăng nhập/gửi OTP/đặt lại mật khẩu (rate limit).
- Thông báo lỗi đăng nhập chung chung, không tiết lộ "email không tồn tại".

## 4. Dữ liệu và cơ sở dữ liệu
```js
// ĐÚNG: tham số hoá
await db.query('SELECT * FROM users WHERE email = $1', [email])
// SAI: ghép chuỗi → SQL injection
await db.query(`SELECT * FROM users WHERE email = '${email}'`)
```
- Bật RLS/Security Rules cho mọi bảng (xem [supabase.md](../02-services/supabase.md), [firebase.md](../02-services/firebase.md)).
- Tài khoản DB của app chỉ có quyền cần thiết, không dùng superuser.
- Mã hoá khi truyền (TLS) và, với dữ liệu rất nhạy cảm, mã hoá cột/trường.
- Sao lưu định kỳ và **thử khôi phục** ít nhất một lần.
- Thu thập ít dữ liệu cá nhân nhất có thể; xoá khi không cần.

## 5. XSS, CSRF, CORS
- **XSS:** framework hiện đại (React, Vue...) escape mặc định; đừng dùng `dangerouslySetInnerHTML`/`v-html` với dữ liệu người dùng. Làm sạch HTML bằng thư viện (DOMPurify).
- **CSRF:** dùng cookie `SameSite`, token CSRF cho form dùng cookie phiên; API dùng header `Authorization` ít bị CSRF hơn.
- **CORS:** chỉ cho phép đúng origin của app. Không đặt `*` cùng `credentials`.

## 6. HTTP header bảo mật
Thêm (qua server, Cloudflare Transform Rules, hoặc file `_headers` của Cloudflare Pages / `firebase.json`):
```
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Content-Type-Options: nosniff
Referrer-Policy: strict-origin-when-cross-origin
Content-Security-Policy: default-src 'self'; frame-ancestors 'none'
Permissions-Policy: camera=(), microphone=(), geolocation=()
```
CSP cần chỉnh theo app (script/style/ảnh từ đâu), thử ở chế độ `Content-Security-Policy-Report-Only` trước. Kiểm tra kết quả bằng ZAP baseline ([security-scanning.md](security-scanning.md)) hoặc công cụ kiểm tra header trực tuyến.

## 7. Upload file, API và thanh toán
- Upload: kiểm tra loại và kích thước ở server, lưu ngoài thư mục phục vụ web (R2/Supabase Storage), đổi tên file, không thực thi file người dùng tải lên.
- API: xác thực + phân quyền từng endpoint, kiểm tra schema đầu vào, giới hạn kích thước body, rate limit.
- Thanh toán: dùng cổng thanh toán (Stripe, VNPay, MoMo...) để **không bao giờ chạm vào số thẻ**; xác minh webhook bằng chữ ký.

## 8. Hạ tầng và tài khoản
- 2FA cho mọi tài khoản dịch vụ; dùng trình quản lý mật khẩu.
- Máy dev: bật mã hoá đĩa (BitLocker trên Windows, FileVault trên macOS), cập nhật hệ điều hành, khoá màn hình.
- Chỉ mở cổng cần thiết; dùng Cloudflare Tunnel thay vì mở cổng router ([cloudflare.md](../02-services/cloudflare.md)).
- Máy chủ tự quản (VPS): đăng nhập SSH bằng key, tắt đăng nhập root và mật khẩu, bật tường lửa, tự động cập nhật bản vá bảo mật.
- Tách môi trường dev/staging/production, dùng secret khác nhau.

## 9. Chuỗi cung ứng phần mềm (dependency)
- Commit file lock (`package-lock.json`, `uv.lock`...) và cài bằng `npm ci`.
- Hạn chế số lượng dependency; kiểm tra package mới (số tải, người bảo trì, ngày cập nhật); coi chừng tên giống package nổi tiếng (typosquatting).
- Chạy quét dependency định kỳ bằng tay ([security-scanning.md](security-scanning.md)).
- Đọc kỹ quyền khi dùng GitHub Action/plugin bên thứ ba, ghim theo commit SHA.

## 10. Ghi log và phát hiện sự cố
- Log: đăng nhập thành công/thất bại, đổi mật khẩu/quyền, thao tác xoá, lỗi 4xx/5xx bất thường. **Không log** mật khẩu, token, số thẻ, dữ liệu cá nhân thô.
- Theo dõi lỗi bằng Sentry hoặc Firebase Crashlytics (gói free có hạn mức, xem trang giá).
- Đặt cảnh báo chi phí đột biến, dấu hiệu bị lạm dụng tài nguyên.

## 11. Khi bị lộ/sự cố (quy trình ngắn)
1. **Rotate ngay** key/token bị lộ (đổi trước, điều tra sau).
2. Thu hồi phiên đăng nhập, đổi mật khẩu liên quan.
3. Xem log để biết phạm vi ảnh hưởng.
4. Vá nguyên nhân gốc, triển khai bản sửa.
5. Thông báo người dùng bị ảnh hưởng nếu có dữ liệu cá nhân (tìm hiểu quy định bảo vệ dữ liệu cá nhân hiện hành tại Việt Nam).
6. Ghi lại bài học vào tài liệu này.

## 12. Checklist 10 điểm tối thiểu
- [ ] HTTPS toàn bộ + HSTS
- [ ] Không secret trong git; có `.env.example`
- [ ] Phân quyền ở server/RLS cho mọi dữ liệu
- [ ] Truy vấn tham số hoá, escape đầu ra
- [ ] Auth dùng dịch vụ có sẵn + 2FA cho tài khoản quản trị
- [ ] Rate limit cho đăng nhập/API nhạy cảm
- [ ] Header bảo mật + CORS đúng origin
- [ ] Dependency có lock và đã quét
- [ ] Có log + theo dõi lỗi
- [ ] Có sao lưu đã thử khôi phục và quy trình rotate key

## Tham khảo
- OWASP Top 10:2025: https://owasp.org/Top10/2025/
- OWASP Cheat Sheet Series: https://cheatsheetseries.owasp.org
