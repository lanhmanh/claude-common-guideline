# GitHub

Cập nhật lần cuối: 2026-11-14

## Dùng để làm gì
Lưu mã nguồn, quản lý branch/PR, issue, host web tĩnh (Pages), lưu secret cho deploy.

## Gói free
Repo public và private không giới hạn số lượng. Một số tính năng (ví dụ Pages cho repo private, một số rule bảo vệ branch) bị giới hạn ở gói free. Xem: https://github.com/pricing

## Setup máy
```powershell
# Windows
winget install Git.Git GitHub.cli
```
```bash
# macOS
brew install git gh
```
```bash
git config --global user.name "Tên bạn"
git config --global user.email "email@example.com"
git config --global init.defaultBranch main
gh auth login
```

## SSH key (tuỳ chọn, thay cho HTTPS token)
```bash
ssh-keygen -t ed25519 -C "email@example.com"
# macOS
pbcopy < ~/.ssh/id_ed25519.pub
# Windows PowerShell
Get-Content $env:USERPROFILE\.ssh\id_ed25519.pub | Set-Clipboard
```
Dán vào GitHub → Settings → SSH and GPG keys. Kiểm tra: `ssh -T git@github.com`.

## Quy trình làm việc cơ bản
```bash
git switch -c feature/ten-tinh-nang
git add -A && git commit -m "feat: mô tả ngắn"
git push -u origin feature/ten-tinh-nang
gh pr create --fill
```

## `.gitignore` tối thiểu
```
.env
.env.*
!.env.example
node_modules/
.venv/
build/
dist/
bin/
obj/
.DS_Store
```

## Secrets
- Không commit key/token. Lỡ commit: **rotate key ngay**, rồi mới dọn lịch sử.
- Secret cho deploy: Settings → Secrets and variables.

## Actions: chỉ chạy tay
Không bật workflow tự chạy theo `push`, `pull_request`, `schedule`. Nếu cần workflow, chỉ dùng:
```yaml
on:
  workflow_dispatch:
```
Chạy bằng nút **Run workflow** hoặc `gh workflow run <tên>`.

## GitHub Pages (web tĩnh)
Settings → Pages → chọn nguồn (branch/thư mục). Repo private cần gói trả phí, xem trang giá.
