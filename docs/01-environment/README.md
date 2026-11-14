# Môi trường phát triển – nguyên tắc chung

Cập nhật lần cuối: 2026-11-14

## Vấn đề cần giải quyết
Mỗi dự án cần version ngôn ngữ/thư viện khác nhau. Cài thẳng một bản vào máy sẽ gây **conflict**. Giải pháp gồm 2 lớp:

1. **Version manager**: cài nhiều version song song, chuyển qua lại dễ dàng.
2. **Môi trường riêng theo dự án**: thư viện cài trong thư mục dự án (venv, `node_modules`, ...), version được **ghi vào file trong repo** để máy nào mở cũng dùng đúng bản.

## Bảng chọn nhanh

| Ngôn ngữ | Windows | macOS | File pin version trong repo |
|---|---|---|---|
| Java | Scoop (`java` bucket) hoặc winget; trỏ `JAVA_HOME` theo dự án | SDKMAN hoặc `brew` + `/usr/libexec/java_home` | `.sdkmanrc` (macOS), `pom.xml`/`build.gradle` toolchain |
| Python | `uv` (khuyên dùng) hoặc `pyenv-win` | `uv` hoặc `pyenv` | `.python-version`, `pyproject.toml` |
| Node.js | `fnm` hoặc `nvm-windows` hoặc Volta | `fnm` hoặc `nvm` hoặc Volta | `.nvmrc` / `.node-version`, `engines` trong `package.json` |
| C# / .NET | Cài nhiều SDK song song (winget) | `brew` hoặc installer chính thức | `global.json` |
| C++ | Visual Studio Build Tools + CMake + vcpkg | Xcode CLT + CMake + brew | `CMakePresets.json`, `vcpkg.json` |

## Cài trình quản lý gói trước
**Windows (PowerShell)**
```powershell
# winget có sẵn trên Windows 10/11 hiện đại
winget --version
# Scoop (cài không cần quyền admin)
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
irm get.scoop.sh | iex
```
**macOS (zsh)**
```bash
xcode-select --install
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

## Quy tắc vàng
- Mỗi dự án có file pin version, **commit vào git**.
- Không `pip install` / `npm install -g` bừa vào hệ thống. Cài vào venv hoặc dùng `npx`/`uvx`.
- Mở dự án mới: kiểm tra version (`node -v`, `python --version`, `java -version`, `dotnet --version`) trước khi cài thư viện.
- Cần cô lập tuyệt đối (khác OS, khác thư viện hệ thống): dùng [Dev Container](devcontainer.md).
