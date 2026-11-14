# Cloudflare

Cập nhật lần cuối: 2026-11-14

## Dùng để làm gì
DNS, HTTPS miễn phí, CDN, **Pages** (host web tĩnh/SSR), **Workers** (serverless), **R2** (lưu file), **Tunnel** (đưa app chạy ở máy nhà ra internet không cần mở port), **Registrar** (mua domain giá gốc).

## Gói free
Có gói Free cho DNS/CDN/SSL cơ bản, Pages, Workers (có hạn mức request/ngày), Tunnel. Xem: https://www.cloudflare.com/plans và https://developers.cloudflare.com/workers/platform/pricing

## 1. Đưa domain về Cloudflare
1. Đăng ký tại https://dash.cloudflare.com.
2. **Add a site** → nhập domain → chọn **Free**.
3. Đổi **nameserver** tại nơi mua domain thành 2 nameserver Cloudflare cung cấp. Chờ vài phút đến vài giờ.
4. **SSL/TLS → Overview**: chọn **Full (strict)** khi origin đã có chứng chỉ hợp lệ.

## 2. Deploy web lên Cloudflare Pages (không cần CI tự động)
```bash
npm install -g wrangler           # hoặc dùng: npx wrangler
wrangler login
npm run build
wrangler pages deploy dist --project-name myapp
```
Gắn domain riêng: Pages → project → **Custom domains**.
> Kết nối GitHub để auto-build mỗi lần push là tuỳ chọn, mặc định không bật. Chỉ bật khi chủ repo yêu cầu rõ.

## 3. Worker đơn giản
```bash
npm create cloudflare@latest my-worker
cd my-worker
npx wrangler dev                  # chạy local
npx wrangler deploy
```

## 4. Tunnel: public app đang chạy local
```powershell
# Windows
winget install Cloudflare.cloudflared
```
```bash
# macOS
brew install cloudflared
```
Thử nhanh, không cần tài khoản/domain (URL tạm `*.trycloudflare.com`):
```bash
cloudflared tunnel --url http://localhost:3000
```
Dùng lâu dài với domain riêng:
```bash
cloudflared tunnel login
cloudflared tunnel create myapp
cloudflared tunnel route dns myapp app.example.com
# file ~/.cloudflared/config.yml
#   tunnel: <ID>
#   credentials-file: <đường dẫn json>
#   ingress:
#     - hostname: app.example.com
#       service: http://localhost:3000
#     - service: http_status:404
cloudflared tunnel run myapp
```
Nếu app có dữ liệu riêng tư, bảo vệ bằng **Cloudflare Access** (Zero Trust, có gói free cho số người dùng nhỏ).

## Lỗi thường gặp
- Lỗi redirect loop: SSL mode đang để Flexible trong khi origin ép HTTPS. Đổi sang Full (strict).
- DNS chưa cập nhật: kiểm tra bằng `nslookup example.com` / `dig example.com NS`.
