# Graphify

Cập nhật lần cuối: **20/11/2026**

## Dùng để làm gì

**Graphify** là công cụ chuyên dùng để **hình dung mã nguồn thành một đồ thị tri thức có thể truy vấn**. Nó phân tích mã từ khoảng 40 ngôn ngữ lập trình bằng tree-sitter AST (xử lý cục bộ, không dùng LLM) và tạo ra:

- **graph.html** – biểu đồ tương tác có thể click được
- **GRAPH_REPORT.md** – báo cáo tóm tắt và các câu hỏi gợi ý
- **graph.json** – đồ thị đầy đủ để truy vấn lập trình

Mục đích: Giúp bạn hiểu cấu trúc dự án, trace kết nối giữa các module, tìm kiếm và phân tích dependencies.

## Cài đặt

### macOS

**Yêu cầu:**
- Python 3.11 trở lên (khuyên Python 3.14+)
- pipx (công cụ cài đặt Python packages cô lập)

**Cài pipx (nếu chưa có):**

```bash
pip3 install pipx
pipx ensurepath
```

Sau khi cài xong, **khởi động lại terminal** hoặc chạy:

```bash
source ~/.zshrc
```

**Cài Graphify:**

```bash
pipx install graphifyy
```

Lệnh này tải package `graphifyy` (tên trên PyPI có hai chữ y), nhưng CLI command là `graphify` (một chữ y).

**Xác nhận cài xong:**

```bash
graphify --version
```

Kết quả sẽ hiển thị version (ví dụ: `graphify 0.9.113`).

---

### Windows

**Yêu cầu:**
- Python 3.11 trở lên (khuyên Python 3.14+) – [Tải từ python.org](https://www.python.org/downloads/)
- Khi cài Python, **tick vào "Add Python to PATH"**

**Cài pipx:**

Mở PowerShell (không cần chạy as Admin):

```powershell
py -m pip install --user pipx
py -m pipx ensurepath
```

Sau khi chạy xong, **khởi động lại PowerShell** để PATH cập nhật.

**Cài Graphify:**

```powershell
pipx install graphifyy
```

**Xác nhận cài xong:**

```powershell
graphify --version
```

Kết quả sẽ hiển thị version.

---

## Cách dùng

### Cơ bản: Phân tích toàn bộ dự án

Vào thư mục dự án và chạy:

**macOS:**
```bash
graphify .
```

**Windows:**
```powershell
graphify .
```

Graphify sẽ:
1. Quét mã nguồn trong folder hiện tại
2. Tạo đồ thị kết nối
3. Sinh ra 3 file: `graph.html`, `GRAPH_REPORT.md`, `graph.json`

Mở `graph.html` bằng trình duyệt để xem biểu đồ tương tác.

### Lệnh thường dùng

**Truy vấn đồ thị (hỏi câu hỏi):**

```bash
graphify query "what connects auth to the database?"
```

**Trace đường dẫn giữa hai module:**

```bash
graphify path "UserService" "DatabasePool"
```

**Giải thích một node (class, function, file) và các kết nối:**

```bash
graphify explain "RateLimiter"
```

**Phân tích lại (nếu code thay đổi):**

```bash
graphify . --update
```

---

## Tích hợp vào Claude Code (AI Assistant)

Nếu dùng Claude Code, bạn có thể cài Graphify làm skill tự động:

**macOS/Windows:**
```bash
graphify install
```

Lệnh này tự nhận diện Claude Code và đăng ký skill `/graphify`. Từ đó trong Claude Code, bạn chỉ cần gõ:

```
/graphify .
```

thay vì gõ toàn bộ command.

---

## Bảo mật

**Code-only analysis (mặc định):** Toàn bộ xử lý diễn ra cục bộ trên máy, không gửi mã lên mạng.

**Nếu dùng docs, PDF, images:** Graphify có thể dùng Claude, OpenAI, Gemini, hoặc LLM khác để phân tích file này. Hãy cấu hình AI backend của bạn trước (xem tài liệu Graphify chính thức).

---

## Lỗi thường gặp

| Lỗi | Giải pháp |
| --- | --- |
| **graphify: command not found** | Chạy `pipx ensurepath` và khởi động lại terminal |
| **Python 3.11+ required** | Cập nhật Python (khuyên v3.14+) rồi cài lại graphifyy |
| **No module named graphifyy** | Cài lại: `pipx uninstall graphifyy && pipx install graphifyy` |
| **graph.html không hiện** | Kiểm tra folder có file `.md`, `.js`, `.py`, `.ts` v.v.; nếu folder trống, Graphify không tạo đồ thị |
| **Permission denied (Windows)** | Chạy PowerShell as Administrator |

---

## Tài liệu chính thức

- Repository: https://github.com/Graphify-Labs/graphify
- Docs: Xem README trong repository

---

## Các bước tiếp theo

1. **Cài xong,** thử: `graphify . --help` để xem tất cả option
2. **Phân tích dự án của bạn:** Vào thư mục code, chạy `graphify .`
3. **Tích hợp vào Claude Code:** Chạy `graphify install` để dùng `/graphify` skill
4. **Truy vấn thông minh:** Dùng `graphify query` để hỏi câu hỏi về cấu trúc code
