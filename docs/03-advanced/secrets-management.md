# Quản lý secret (API key, mật khẩu DB, token)

Cập nhật lần cuối: 2026-11-14

## Cấp độ tăng dần

| Cấp | Cách | Hợp với |
|---|---|---|
| 0 | `.env` local + `.gitignore` | Một người, chạy local |
| 1 | Secret của nền tảng deploy (Cloudflare, Firebase, GitHub...) | App nhỏ đã public |
| 2 | Secret manager dạng SaaS/self-host (Infisical, Doppler) | Nhiều môi trường, nhiều người |
| 3 | Vault / OpenBao tự vận hành | Hệ thống lớn, cần rotate, dynamic secret, audit |

Luôn bắt đầu từ cấp thấp nhất đủ dùng.

## Cấp 0: `.env` đúng cách
- Commit `.env.example` (khoá có, giá trị trống). **Không** commit `.env`.
- Bật quét secret trước khi commit:
```bash
# macOS
brew install gitleaks
# Windows
winget install Gitleaks.Gitleaks
gitleaks detect --source .        # quét repo hiện tại
```
- GitHub có *push protection* chặn push khi phát hiện secret phổ biến, nên bật trong Settings → Code security.
- Lỡ commit secret: **rotate (đổi) key ngay**, sau đó mới dọn lịch sử. Xoá khỏi git không làm key hết hiệu lực.

## Cấp 1: Secret của nền tảng
```bash
npx wrangler secret put API_KEY            # Cloudflare Workers
firebase functions:secrets:set API_KEY     # Firebase Functions
gh secret set API_KEY                      # GitHub Actions/repo secrets
```
Phân biệt khoá **public** (đưa vào client được, như Supabase anon key, Firebase web apiKey) và khoá **private** (chỉ server, như service_role, Client Secret).

## Cấp 2: Secret manager
- **Infisical**: mã nguồn mở, tự host được, có gói cloud free cho team nhỏ.
- **Doppler**: SaaS, cài nhanh, có gói free cho cá nhân/team nhỏ.
- **SOPS + age**: mã hoá file secret rồi commit vào git, không cần server. Hợp GitOps, một đến vài người.
- **1Password / Bitwarden Secrets Manager**: nếu đã dùng sẵn.

Giới hạn gói free (số user, audit log, SSO) thay đổi và thường bị chặn ở gói trả phí, xem trang giá từng hãng trước khi chọn.

Ví dụ SOPS + age:
```bash
brew install sops age              # Windows: winget install Mozilla.SOPS FiloSottile.age
age-keygen -o ~/.config/sops/age/keys.txt
sops --encrypt --age <public-key> secrets.yaml > secrets.enc.yaml
sops --decrypt secrets.enc.yaml
```
Khoá bí mật `keys.txt` không bao giờ commit; sao lưu riêng.

## Cấp 3: HashiCorp Vault và OpenBao
Điểm cần biết khi đánh giá (đã kiểm tra tháng 11/2026):
- **HCP Vault Secrets** (bản managed rẻ) đã ngừng bán từ 30/06/2025 và hết vòng đời vào 01/07/2026. Tài liệu/blog cũ nhắc tới nó là lỗi thời. Nguồn: https://developer.hashicorp.com/hcp/docs/vault-secrets/end-of-sale-announcement
- Vault Community vẫn tự host miễn phí nhưng giấy phép là **BSL 1.1** (không phải mã nguồn mở thuần) từ 2023. Cần xem điều khoản nếu dùng thương mại.
- **OpenBao**: bản fork mã nguồn mở (MPL 2.0, thuộc Linux Foundation), API tương thích Vault, tự vận hành, không có dịch vụ managed chính thức.
- Managed của HashiCorp còn lại là HCP Vault Dedicated (tính phí theo cluster), thường quá đắt cho dự án nhỏ.

Chạy thử Vault dev mode bằng Docker (**chỉ để học, dữ liệu mất khi tắt**):
```bash
docker run --rm -p 8200:8200 -e VAULT_DEV_ROOT_TOKEN_ID=dev-only hashicorp/vault
export VAULT_ADDR=http://127.0.0.1:8200 VAULT_TOKEN=dev-only
vault kv put secret/myapp db_password=changeme
vault kv get secret/myapp
```
Đưa vào production cần: storage backend bền (Raft), TLS, **unseal/auto-unseal**, policy theo nguyên tắc tối thiểu, bật audit log, sao lưu snapshot, kế hoạch xoay khoá. Đây là việc vận hành thật sự, cần người phụ trách.

Nếu hạ tầng đặt tại nhà cung cấp trong nước, thường phải tự chạy Vault/OpenBao trên VM (2 đến 3 node cho HA). Xem danh mục dịch vụ hiện hành của nhà cung cấp trước khi giả định họ có secret manager managed.

## Checklist
- [ ] Không có secret trong git (đã chạy gitleaks)
- [ ] `.env.example` có mặt, `.env` trong `.gitignore`
- [ ] Mỗi môi trường (dev/staging/prod) dùng secret khác nhau
- [ ] Có quy trình rotate khi nhân sự thay đổi hoặc nghi lộ
- [ ] Khoá private không nằm trong app client/mobile
