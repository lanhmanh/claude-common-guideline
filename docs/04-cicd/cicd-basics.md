# CI/CD cơ bản

Cập nhật lần cuối: 2026-11-14

> **Quy ước của repo này:** không bật CI/build tự động (trigger `push`, `pull_request`, `schedule`). Pipeline chỉ chạy **bằng tay** (`workflow_dispatch` hoặc lệnh CLI) khi chủ repo yêu cầu rõ. Tài liệu này giải thích khái niệm để hiểu và chủ động chọn khi cần, không phải để bật tự động.

## CI và CD là gì
| Thuật ngữ | Ý nghĩa | Ví dụ |
|---|---|---|
| **CI** (Continuous Integration) | Mỗi thay đổi code được tự động build + test để phát hiện lỗi sớm | `npm ci && npm test` |
| **CD** (Continuous Delivery) | Bản build đã qua kiểm tra luôn sẵn sàng phát hành; bấm nút là deploy | Artifact + nút "Deploy" |
| **CD** (Continuous Deployment) | Qua kiểm tra là tự động lên production, không cần người duyệt | Merge là lên live |

Với dự án cá nhân/nhỏ, thực tế hay dùng: **chạy test + build bằng tay trên máy**, rồi deploy bằng CLI. Chỉ cần máy chủ CI khi nhiều người cùng sửa hoặc cần build môi trường mà máy mình không có (ví dụ build iOS khi không có Mac).

## Các khái niệm chính
- **Pipeline / Workflow**: chuỗi bước tự động (cài thư viện → lint → test → build → deploy).
- **Job**: một nhóm bước chạy trên một máy. Nhiều job có thể chạy song song.
- **Runner / Agent**: máy thực thi job. *Hosted* (nhà cung cấp lo) hoặc *self-hosted* (máy của bạn).
- **Trigger**: sự kiện kích hoạt (push, PR, lịch, bấm tay).
- **Artifact**: file kết quả của build (APK, `dist/`, báo cáo test).
- **Cache**: lưu thư viện đã tải để lần sau chạy nhanh hơn (tiết kiệm phút build = tiết kiệm tiền).
- **Secret**: biến bí mật cho pipeline (token deploy). Xem [secrets-management.md](../03-advanced/secrets-management.md).
- **Environment**: dev / staging / production, mỗi nơi có cấu hình và secret riêng.
- **Matrix**: chạy cùng job trên nhiều tổ hợp (Node 20/22, Windows/macOS). Mỗi tổ hợp tốn thêm phút build.

## Một pipeline điển hình
```
code → lint → test → build → (artifact) → deploy staging → kiểm tra → deploy production
```
Nguyên tắc:
1. **Build một lần, deploy nhiều lần**: cùng một artifact đi qua các môi trường.
2. Pipeline là **code trong repo** (YAML), có review như code khác.
3. Chạy được ở máy local bằng đúng các lệnh đó, pipeline chỉ là tự động hoá.
4. Có cách **rollback** (giữ bản cũ, deploy lại bản trước).
5. Pipeline nhanh (vài phút), nếu không sẽ bị bỏ qua.

## Chạy tay thay cho tự động
Mọi bước nên có lệnh chạy tay, ví dụ ghi trong `package.json`:
```json
{
  "scripts": {
    "check": "npm run lint && npm test && npm run build",
    "deploy:web": "npx wrangler pages deploy dist --project-name myapp"
  }
}
```
```bash
npm run check && npm run deploy:web
```
Khi muốn chạy trên máy chủ CI nhưng vẫn do bạn quyết định: dùng trigger bấm tay.

**GitHub Actions – chỉ bấm tay**
```yaml
# .github/workflows/manual-check.yml
name: manual-check
on:
  workflow_dispatch:        # KHÔNG thêm push / pull_request / schedule
jobs:
  check:
    runs-on: ubuntu-latest
    timeout-minutes: 15
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version-file: .node-version, cache: npm }
      - run: npm ci
      - run: npm run check
```
Chạy: tab **Actions → manual-check → Run workflow**, hoặc `gh workflow run manual-check`.
Đặt `timeout-minutes` để job treo không ăn hết số phút miễn phí.

## Chiến lược deploy (khái niệm)
| Chiến lược | Ý tưởng | Khi dùng |
|---|---|---|
| Thay thế trực tiếp | Ghi đè bản cũ bằng bản mới | App nhỏ, chấp nhận gián đoạn ngắn |
| Blue/green | Hai môi trường song song, chuyển lưu lượng | Cần rollback tức thì |
| Canary | Cho một phần nhỏ người dùng dùng bản mới trước | Hệ thống lớn |
| Feature flag | Code đã lên nhưng tính năng tắt/bật bằng cờ | Phát hành dần |
| Preview deploy | Mỗi nhánh/PR có URL xem thử riêng | Review giao diện |

Hosting tĩnh như Cloudflare Pages/Firebase Hosting đã giữ lịch sử bản deploy và cho rollback trên dashboard.

## Bảo mật pipeline
- Secret chỉ cấp cho job cần dùng, không in ra log.
- Ghim version của action/plugin bên thứ ba (theo tag hoặc commit SHA), vì đây là code chạy với quyền của bạn.
- Quyền token tối thiểu (`permissions:` trong GitHub Actions).
- Với repo public: PR từ fork không được tiếp cận secret, đừng nới điều này.
- Dùng OIDC để lấy credential tạm thời của cloud thay vì lưu key dài hạn, nếu nhà cung cấp hỗ trợ.

## Bước tiếp theo
Chọn công cụ và ước tính chi phí: [cicd-tools-and-costs.md](cicd-tools-and-costs.md).
