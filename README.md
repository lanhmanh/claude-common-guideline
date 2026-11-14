# claude-common-guideline

Kho tài liệu **research + hướng dẫn kiến thức IT dùng chung**, đi từ cơ bản đến nâng cao, phục vụ việc dựng một application từ lúc chạy **local** đến khi **public**. Tài liệu được cập nhật liên tục theo các dịch vụ/công cụ mới (xem [CONTRIBUTING.md](CONTRIBUTING.md)).

> Ngôn ngữ: tiếng Việt. Hỗ trợ **Windows** và **macOS**.

## Lộ trình tổng quát

```
Cài môi trường  →  Code local  →  Container hóa  →  Backend/DB/Auth  →  Deploy  →  Domain + HTTPS  →  Public
   (01)             (01)           (02/docker)      (02/supabase,       (02/cloudflare, (02/cloudflare)
                                                      firebase,           firebase)
                                                      google-oauth)
```

## Mục lục

### 01 – Môi trường phát triển
| Tài liệu | Nội dung |
|---|---|
| [01-environment/README.md](docs/01-environment/README.md) | Nguyên tắc chung, cách tránh conflict giữa các dự án |
| [java.md](docs/01-environment/java.md) | Đổi version Java, môi trường riêng từng dự án (Win/macOS) |
| [python.md](docs/01-environment/python.md) | Đổi version Python, virtualenv / uv (Win/macOS) |
| [node.md](docs/01-environment/node.md) | Đổi version Node.js, pin version theo dự án (Win/macOS) |
| [dotnet.md](docs/01-environment/dotnet.md) | .NET / C#: nhiều SDK song song, `global.json` (Win/macOS) |
| [cpp.md](docs/01-environment/cpp.md) | C++: toolchain, CMake, vcpkg/Conan (Win/macOS) |
| [devcontainer.md](docs/01-environment/devcontainer.md) | Cách cô lập tuyệt đối bằng Dev Container |

### 02 – Dịch vụ thông dụng (ưu tiên gói free)
| Tài liệu | Dùng để |
|---|---|
| [docker.md](docs/02-services/docker.md) | Container hóa app, chạy DB/service local |
| [supabase.md](docs/02-services/supabase.md) | Postgres + Auth + Storage (BaaS) |
| [firebase.md](docs/02-services/firebase.md) | Hosting, Auth, Firestore, push notification |
| [cloudflare.md](docs/02-services/cloudflare.md) | DNS, HTTPS, Pages, Workers, Tunnel |
| [google-oauth.md](docs/02-services/google-oauth.md) | Google Client ID / "Đăng nhập bằng Google" |
| [github.md](docs/02-services/github.md) | Repo, SSH key, secrets, Pages (không bật CI tự động) |

### 03 – Nâng cao
| Tài liệu | Nội dung |
|---|---|
| [secrets-management.md](docs/03-advanced/secrets-management.md) | `.env` → secret manager → Vault/OpenBao, quét secret |
| [domain-vn.md](docs/03-advanced/domain-vn.md) | Mua domain, đăng ký `.vn`, DNS, email theo domain |
| [mobile-release.md](docs/03-advanced/mobile-release.md) | Expo/EAS, Google Play, App Store, chi phí tài khoản |
| [go-live-checklist.md](docs/03-advanced/go-live-checklist.md) | Checklist bảo mật, vận hành, pháp lý trước khi public |

### 04 – CI/CD
| Tài liệu | Nội dung |
|---|---|
| [cicd-basics.md](docs/04-cicd/cicd-basics.md) | Khái niệm CI/CD, pipeline, chiến lược deploy, bảo mật, chạy tay (`workflow_dispatch`) |
| [cicd-tools-and-costs.md](docs/04-cicd/cicd-tools-and-costs.md) | So sánh GitHub Actions, GitLab, CircleCI, Cloudflare, Jenkins...; chi phí và cách chọn |

### 05 – Bảo mật
| Tài liệu | Nội dung |
|---|---|
| [security-basics.md](docs/05-security/security-basics.md) | OWASP Top 10:2025, xác thực, dữ liệu, header, hạ tầng, xử lý sự cố, checklist 10 điểm |
| [security-scanning.md](docs/05-security/security-scanning.md) | Quét secret/SCA/SAST/DAST/container, lệnh chạy tay, dịch vụ GitHub/Snyk/Cloudflare và chi phí |

## Quy ước chung

- Mọi lệnh có phần **Windows (PowerShell)** và **macOS (zsh)** riêng.
- Không commit secret (`.env`, key, token). Luôn có `.env.example`.
- **Không bật CI/build tự động** (push, pull_request, schedule). Workflow chỉ chạy tay (`workflow_dispatch`).
- Giới hạn gói free của dịch vụ thay đổi thường xuyên: mỗi tài liệu có mục *Cập nhật lần cuối* và link tới trang giá chính thức, hãy kiểm tra lại trước khi dựa vào.
