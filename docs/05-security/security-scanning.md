# Quét bảo mật sản phẩm tự động

Cập nhật lần cuối: 2026-11-14

> **Quy ước của repo này:** không bật quét tự động theo trigger `push`, `pull_request`, `schedule`. Mọi cách quét dưới đây mặc định **chạy tay** (lệnh CLI trên máy, hoặc workflow `workflow_dispatch`). "Tự động" ở đây nghĩa là *công cụ tự phân tích*, không phải *tự chạy mỗi lần push*. Chỉ bật trigger tự động khi chủ repo yêu cầu rõ.

## Các loại quét

| Loại | Quét cái gì | Khi nào | Công cụ miễn phí tiêu biểu |
|---|---|---|---|
| **Secret scanning** | Key, token lỡ commit (kể cả lịch sử git) | Trước khi push/public | Gitleaks, TruffleHog |
| **SCA** (Software Composition Analysis) | Lỗ hổng đã biết trong thư viện phụ thuộc | Trước release, định kỳ | OSV-Scanner, Trivy, Grype, `npm audit`, `pip-audit` |
| **SAST** (Static Application Security Testing) | Lỗi bảo mật trong **mã nguồn của bạn** | Trong lúc dev, trước release | Semgrep CE, CodeQL (repo public), SonarCloud |
| **IaC / Dockerfile** | Cấu hình sai trong Terraform, Dockerfile, K8s | Khi sửa hạ tầng | Trivy config, Checkov, Hadolint |
| **Container scan** | Lỗ hổng trong image Docker | Trước khi đẩy image | Trivy, Grype |
| **DAST** (Dynamic) | Tấn công thử app **đang chạy** | Trước go-live, trên staging | ZAP (baseline/full) |
| **Mobile** | Kiểm tra APK/IPA | Trước khi lên store | MobSF |

Bộ khởi đầu miễn phí hay được khuyên dùng: Semgrep CE (SAST), Trivy và Grype (SCA/container), Checkov (IaC), Gitleaks (secret), ZAP (DAST). Các công cụ mã nguồn mở này chạy cục bộ nên không giới hạn số lần quét.

## Cài công cụ

**macOS**
```bash
brew install gitleaks trivy osv-scanner semgrep hadolint
```
**Windows (PowerShell)**
```powershell
winget install Gitleaks.Gitleaks
scoop install trivy osv-scanner hadolint     # hoặc tải bản phát hành trên GitHub của từng tool
# Semgrep: cài trong venv Python
uv tool install semgrep                       # hoặc: pip install semgrep (trong venv)
```
Kiểm tra tên gói hiện hành bằng `winget search` / `scoop search`; tên có thể đổi.

## Lệnh chạy tay thường dùng

### 1. Secret scanning
```bash
gitleaks detect --source . --verbose          # quét repo + lịch sử (bản mới có thể dùng: gitleaks git)
gitleaks dir .                                # chỉ quét thư mục hiện tại (bản mới)
```
Phát hiện secret: **rotate key trước**, rồi mới dọn lịch sử.
Dùng làm pre-commit hook cục bộ (chạy trên máy bạn, không phải CI):
```bash
# .git/hooks/pre-commit (chmod +x)
#!/bin/sh
gitleaks protect --staged --redact
```
(Lệnh con có thể đổi giữa các phiên bản gitleaks; xem `gitleaks --help`.)

### 2. SCA – lỗ hổng trong dependency
```bash
osv-scanner scan source -r .                  # bản cũ: osv-scanner -r .
trivy fs .                                    # quét thư mục (lockfile, secret, cấu hình)
npm audit                                     # Node
pip install pip-audit && pip-audit            # Python (trong venv)
dotnet list package --vulnerable --include-transitive   # .NET
```
Cách đọc kết quả: ưu tiên mục **Critical/High** có bản vá, và mục nằm trên đường chạy thật (không chỉ devDependency).

### 3. SAST – mã nguồn của bạn
```bash
semgrep scan --config auto .
```
`--config auto` tải bộ luật cộng đồng phù hợp ngôn ngữ; tìm hiểu cách Semgrep xử lý số liệu sử dụng nếu cần, và chọn `--metrics=off` khi muốn tắt. Kết quả SAST có **dương tính giả**: xem xét từng cái, không sửa mù quáng.

### 4. Dockerfile, IaC, image
```bash
hadolint Dockerfile
trivy config .                                # Terraform, Dockerfile, K8s...
trivy image myapp:latest                      # quét image đã build
```

### 5. DAST – quét app đang chạy
Chỉ quét hệ thống **của bạn hoặc được phép**. Quét thử trên staging/local, không quét dịch vụ của người khác.
```bash
docker run --rm -t ghcr.io/zaproxy/zaproxy:stable \
  zap-baseline.py -t http://host.docker.internal:3000
```
`zap-baseline.py` chạy quét thụ động (an toàn, nhanh). Quét chủ động (`zap-full-scan.py`) gửi request tấn công thật, **không chạy trên production** và có thể làm đầy dữ liệu test.
Linux/WSL: thay `host.docker.internal` bằng IP phù hợp hoặc `--network host`.

### 6. Mobile
MobSF (mã nguồn mở) chạy bằng Docker: `docker run -it --rm -p 8000:8000 opensecurity/mobile-security-framework-mobsf:latest`, mở `http://localhost:8000` rồi tải APK/IPA lên.

## Quét "trên cloud" mà vẫn chạy tay

### GitHub Actions bấm tay
```yaml
# .github/workflows/manual-security-scan.yml
name: manual-security-scan
on:
  workflow_dispatch:            # KHÔNG thêm push / pull_request / schedule
jobs:
  scan:
    runs-on: ubuntu-latest
    timeout-minutes: 15
    steps:
      - uses: actions/checkout@v4
      - name: Trivy (dependency + secret + config)
        run: |
          curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b /usr/local/bin
          trivy fs --scanners vuln,secret,misconfig --exit-code 0 .
```
Cài tool bằng script tải từ mạng chạy với quyền của job; để chặt chẽ hơn, ghim phiên bản và kiểm tra checksum, hoặc dùng action chính thức của hãng và ghim theo commit SHA. Phút chạy tính vào hạn mức ([cicd-tools-and-costs.md](../04-cicd/cicd-tools-and-costs.md)).

### Dịch vụ GitHub (cần biết chi phí)
| Tính năng | Repo public | Repo private |
|---|---|---|
| Code scanning (CodeQL) | Miễn phí | Cần GitHub Code Security (trả phí) |
| Secret scanning + push protection | Miễn phí | Cần GitHub Secret Protection (trả phí) |
| Dependency review | Miễn phí | Cần Code Security |
| Dependabot alerts | Miễn phí mọi gói | Miễn phí mọi gói |

Giá tham khảo (theo người commit/tháng, kiểm tra lại tại https://github.com/security/plans): Secret Protection khoảng 19 USD, Code Security khoảng 30 USD, cả hai khoảng 49 USD. CodeQL ở repo private còn tiêu tốn phút GitHub Actions. Với cá nhân dùng repo private bản Free, đây thường **không đáng**: dùng Gitleaks + Semgrep + Trivy chạy cục bộ thay thế.

**Dependabot:**
- *Alerts* (cảnh báo lỗ hổng dependency) thụ động, miễn phí, nên bật trong Settings → Code security.
- *Security updates / Version updates* tự mở PR cập nhật thư viện, **đây là hành vi tự động**, chỉ bật khi chủ repo yêu cầu rõ.
- Mặc định của repo này: **không** thêm `.github/dependabot.yml` và không bật các mục tự động.

## Dịch vụ quét bên thứ ba (SaaS) và gói free

| Dịch vụ | Phạm vi | Gói free (kiểm tra lại) |
|---|---|---|
| **Snyk** | SCA, SAST, Container, IaC | Free: 200 test Open Source, 100 test Code, 100 Container, 300 IaC mỗi tháng, không giới hạn developer; Team từ 25 USD/tháng |
| **SonarCloud** | Chất lượng + bảo mật mã | Free tier cho khoảng 50K dòng code, 5 user (một số nguồn) |
| **Semgrep (AppSec Platform)** | SAST, SCA, secret | Có gói Team free cho nhóm nhỏ (một số nguồn nêu tới 10 contributor, 10 repo private) |
| **GitGuardian** | Secret scanning | Có gói free |
| **Socket** | Chuỗi cung ứng npm/PyPI (typosquatting, malware) | Có gói free cho dự án nhỏ |
| **Cloudflare (WAF)** | Chặn tấn công ở rìa mạng | Free: DDoS không giới hạn, WAF Free Managed Ruleset, 5 custom rule; Pro khoảng 20 USD/tháng nếu trả theo năm (Managed Rules đầy đủ) |

Lưu ý: số test của Snyk bị tính theo mỗi lần quét nên **tự động hoá hay quét theo PR sẽ cạn nhanh** (ví dụ 10 repo quét mỗi ngày làm hết 100 test Code trong khoảng 10 ngày). Dùng chạy tay hoặc quét theo đợt trước release để tiết kiệm.

## Quy trình đề xuất (nhẹ, chạy tay)

**Hằng ngày/khi dev (máy local):**
- `gitleaks protect --staged` trong pre-commit hook.
- Plugin Semgrep/Snyk trong IDE (nếu dùng).

**Trước mỗi release (chạy tay, vài phút):**
```bash
gitleaks detect --source .
osv-scanner scan source -r .
semgrep scan --config auto .
trivy fs --scanners vuln,secret,misconfig .
hadolint Dockerfile && trivy image myapp:latest   # nếu có Docker
```

**Trước go-live lần đầu:**
- ZAP baseline trên staging.
- Rà [security-basics.md](security-basics.md) checklist 10 điểm và [go-live-checklist.md](../03-advanced/go-live-checklist.md).

**Định kỳ (đặt nhắc lịch cá nhân, không dùng `schedule`):**
- Mỗi tháng: `osv-scanner`, `npm audit`, cập nhật dependency, xem Dependabot alerts.
- Mỗi quý: thử khôi phục sao lưu, rà tài khoản/quyền, rotate key quan trọng.

## Cách xử lý kết quả
1. **Phân loại:** mức độ (Critical/High/Medium/Low), có khai thác được không, nằm ở code chạy thật hay chỉ dev/test.
2. **Sửa trước** những lỗi: lộ secret, lỗ hổng có khai thác công khai, lỗi ở endpoint công khai.
3. **Dương tính giả:** ghi lý do bỏ qua (file ignore của từng tool) thay vì tắt luật toàn cục.
4. **Không có bản vá:** tìm thư viện thay thế, giảm bề mặt tiếp xúc, hoặc giảm thiểu bằng cấu hình.
5. Ghi lại các lần quét và quyết định (một dòng trong `SECURITY.md` hoặc tài liệu dự án).

## Mẫu `SECURITY.md` cho repo public
```markdown
# Security Policy
Báo lỗ hổng riêng tư qua: security@example.com (hoặc GitHub Security Advisories).
Vui lòng không công khai trước khi chúng tôi có bản vá. Phản hồi dự kiến trong 7 ngày.
```

## Tham khảo
- GitHub security features: https://docs.github.com/en/code-security/getting-started/github-security-features
- Gói bảo mật GitHub: https://github.com/security/plans
- Snyk plans: https://snyk.io/plans
- Trivy: https://trivy.dev · OSV-Scanner: https://google.github.io/osv-scanner · Semgrep: https://semgrep.dev · Gitleaks: https://github.com/gitleaks/gitleaks · ZAP: https://www.zaproxy.org
