# Google OAuth (Google Client ID) – "Đăng nhập bằng Google"

Cập nhật lần cuối: 2026-11-14

## Dùng để làm gì
Cho người dùng đăng nhập app bằng tài khoản Google. Cần **OAuth Client ID** (và Client Secret cho web server). Tạo Client ID **miễn phí**.

## Setup trên Google Cloud Console
1. Vào https://console.cloud.google.com, tạo **New Project**.
2. Mở **Google Auth Platform** (hoặc **APIs & Services → OAuth consent screen** tuỳ giao diện hiện tại).
3. Cấu hình **consent screen**: tên app, email hỗ trợ, domain, loại **External**.
4. Trạng thái **Testing**: chỉ các tài khoản thêm vào *Test users* mới đăng nhập được. Muốn mọi người dùng được thì **Publish app** (scope cơ bản như email/profile thường không cần xác minh nặng; scope nhạy cảm thì cần Google review).
5. **Credentials → Create credentials → OAuth client ID**, chọn loại:

| Loại app | Chọn | Cần điền |
|---|---|---|
| Web | Web application | Authorized JavaScript origins, Authorized redirect URIs |
| Android | Android | package name + SHA-1 của keystore |
| iOS | iOS | Bundle ID |
| Desktop | Desktop app | không cần URI |

## Giá trị URI điển hình
- Local web: origin `http://localhost:3000`, redirect `http://localhost:3000/auth/callback`
- Production: `https://app.example.com`, `https://app.example.com/auth/callback`
- Dùng Supabase: redirect là `https://<ref>.supabase.co/auth/v1/callback` (copy từ dashboard Supabase)
- Dùng Firebase Auth: Firebase tự tạo, chỉ cần bật provider Google

## Lấy SHA-1 cho Android
```bash
# macOS
keytool -list -v -keystore ~/.android/debug.keystore -alias androiddebugkey -storepass android -keypass android
```
```powershell
# Windows
keytool -list -v -keystore $env:USERPROFILE\.android\debug.keystore -alias androiddebugkey -storepass android -keypass android
```
Bản release cần SHA-1 của keystore release (và của Play App Signing nếu dùng).

## Lưu trong app
```
GOOGLE_CLIENT_ID=xxxx.apps.googleusercontent.com
GOOGLE_CLIENT_SECRET=...      # chỉ server, không đưa vào client/mobile
```
Commit `.env.example` với giá trị trống, không commit `.env`.

## Lỗi thường gặp
- `redirect_uri_mismatch`: URI trong request phải khớp **từng ký tự** với URI đã khai báo (http/https, dấu `/` cuối, port).
- `access_blocked` / app chưa xác minh: app đang ở Testing và email chưa nằm trong Test users.
- Thay đổi cấu hình chưa có hiệu lực: chờ vài phút.
