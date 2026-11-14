# Node.js – đổi version và cô lập theo dự án

Cập nhật lần cuối: 2026-11-14

Node cô lập thư viện sẵn bằng `node_modules` trong từng dự án. Việc cần làm là **pin version Node** theo dự án. Xem bản LTS hiện hành tại https://nodejs.org.

## Cách khuyên dùng: fnm (nhanh, chạy cả Win + macOS)

**Cài**
```powershell
# Windows
winget install Schniz.fnm
```
```bash
# macOS
brew install fnm
```

**Bật tự động đổi version khi `cd`**
```bash
# macOS: thêm vào ~/.zshrc
eval "$(fnm env --use-on-cd --shell zsh)"
```
```powershell
# Windows: thêm vào $PROFILE (mở bằng: notepad $PROFILE)
fnm env --use-on-cd --shell powershell | Out-String | Invoke-Expression
```

**Dùng**
```bash
fnm list-remote | tail
fnm install --lts
fnm install 20
fnm use 20
node -v

echo "20" > .node-version     # pin cho dự án, fnm tự đọc khi cd vào
```

## Các lựa chọn khác
- **nvm** (macOS/Linux): `curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/master/install.sh | bash`, rồi `nvm install 20`, `nvm use`, pin bằng `.nvmrc`.
- **nvm-windows** (Windows, dự án riêng, khác nvm): https://github.com/coreybutler/nvm-windows. `nvm install 20.18.0`, `nvm use 20.18.0` (cần quyền admin khi `use`).
- **Volta** (Win + macOS): `volta install node@20`, `volta pin node@20` ghi vào `package.json`, tự đổi version theo dự án.

## Pin version trong dự án
```json
// package.json
{ "engines": { "node": ">=20 <21" } }
```
Bật Corepack để pin cả package manager:
```bash
corepack enable
corepack use pnpm@latest      # ghi "packageManager" vào package.json
```

## Cài thư viện đúng cách
```bash
npm ci            # dựng chính xác theo package-lock.json (máy mới, deploy)
npm install       # khi thêm/sửa dependency
npx <tool>        # chạy tool không cần cài global
```
Không commit `node_modules/`. Luôn commit file lock.

## Lỗi thường gặp
- Windows: `fnm` không đổi version sau khi cd: chưa thêm dòng vào `$PROFILE`, hoặc chưa mở terminal mới.
- Gõ `node -v` ra bản cũ: còn bản cài trực tiếp từ nodejs.org nằm trước trong `Path`. Gỡ bản đó đi.
- Lỗi native module sau khi đổi version: xoá `node_modules` rồi `npm ci`.
