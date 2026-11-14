# Python – đổi version và môi trường ảo

Cập nhật lần cuối: 2026-11-14

Quy tắc: **mỗi dự án một virtualenv (`.venv`)**, và version Python ghi trong `.python-version`.

## Cách khuyên dùng: uv (Win + macOS, một công cụ cho tất cả)
`uv` quản lý cả version Python, venv và thư viện. Tài liệu: https://docs.astral.sh/uv/

**Cài**
```powershell
# Windows
winget install --id=astral-sh.uv -e
```
```bash
# macOS
brew install uv
```

**Dùng**
```bash
uv python list                  # xem version có thể cài
uv python install 3.12 3.13     # cài nhiều version
uv python pin 3.12              # ghi .python-version cho dự án

uv venv                         # tạo .venv theo version đã pin
uv pip install requests         # cài vào .venv

# Hoặc theo luồng dự án có pyproject.toml
uv init myapp && cd myapp
uv add fastapi uvicorn
uv run python main.py           # tự dùng đúng venv
uv sync                         # dựng lại môi trường từ uv.lock trên máy khác
```

## Cách truyền thống: pyenv + venv

**macOS**
```bash
brew install pyenv
echo 'eval "$(pyenv init -)"' >> ~/.zshrc && source ~/.zshrc
pyenv install --list | grep " 3\.12"
pyenv install 3.12.8
pyenv local 3.12.8              # ghi .python-version trong thư mục dự án
python -m venv .venv
source .venv/bin/activate
```

**Windows (PowerShell)** – dùng pyenv-win hoặc Python Launcher `py`
```powershell
# pyenv-win
winget install pyenv-win.pyenv-win   # hoặc: scoop install pyenv
pyenv install 3.12.8
pyenv local 3.12.8

# hoặc py launcher (đi kèm bản cài chính thức từ python.org)
py -0                      # liệt kê version đã cài
py -3.12 -m venv .venv
.\.venv\Scripts\Activate.ps1
```
Nếu PowerShell chặn script: `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned`.

## Chốt thư viện để dựng lại
```bash
pip freeze > requirements.txt     # hoặc dùng uv.lock / poetry.lock
pip install -r requirements.txt
```
Thêm `.venv/` vào `.gitignore`.

## Lỗi thường gặp
- `pip` cài ra ngoài venv: chưa activate. Kiểm tra `which python` / `where python`.
- macOS báo `externally-managed-environment`: đang dùng Python hệ thống/Homebrew, hãy tạo venv rồi cài trong đó.
- Windows mở Microsoft Store khi gõ `python`: tắt App execution alias (Settings → Apps → Advanced app settings) hoặc dùng `py`.
