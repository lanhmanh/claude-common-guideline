# Supabase

Cập nhật lần cuối: 2026-11-14

## Dùng để làm gì
Backend sẵn có: **Postgres**, **Auth** (email, OAuth Google...), **Storage**, **Realtime**, **Edge Functions**. Hợp cho app nhỏ đến vừa, không muốn tự dựng server.

## Gói free
Có gói Free với giới hạn về dung lượng DB, storage, băng thông, số người dùng active. **Dự án free không hoạt động một thời gian sẽ bị tạm dừng (pause)**. Số liệu cụ thể thay đổi, xem: https://supabase.com/pricing

## Setup trên web
1. Đăng ký tại https://supabase.com (đăng nhập bằng GitHub).
2. **New project**: đặt tên, đặt mật khẩu DB (lưu vào password manager), chọn region gần người dùng (Singapore cho Việt Nam).
3. Vào **Project Settings → API** lấy:
   - `Project URL`
   - `anon` / publishable key: dùng được ở client, **bảo mật bằng RLS**.
   - `service_role` / secret key: **chỉ dùng ở server**, tuyệt đối không đưa vào app client.

## Kết nối vào app (JS)
```bash
npm install @supabase/supabase-js
```
`.env`
```
SUPABASE_URL=https://xxxx.supabase.co
SUPABASE_ANON_KEY=...
```
```js
import { createClient } from '@supabase/supabase-js'
const supabase = createClient(process.env.SUPABASE_URL, process.env.SUPABASE_ANON_KEY)
const { data, error } = await supabase.from('todos').select('*')
```

## Bảo mật bắt buộc: Row Level Security
```sql
create table todos (
  id bigint generated always as identity primary key,
  user_id uuid not null default auth.uid(),
  title text not null
);
alter table todos enable row level security;
create policy "owner read" on todos for select using (auth.uid() = user_id);
create policy "owner write" on todos for insert with check (auth.uid() = user_id);
```
Bảng không bật RLS mà lộ anon key là ai cũng đọc/ghi được.

## Đăng nhập Google
Dashboard → **Authentication → Providers → Google**, dán Client ID/Secret lấy từ [google-oauth.md](google-oauth.md), copy **Callback URL** của Supabase dán ngược vào mục Authorized redirect URIs bên Google.

## Chạy local bằng CLI (cần [Docker](docker.md))
```bash
npm install -g supabase       # hoặc: brew install supabase/tap/supabase | scoop install supabase
supabase init
supabase start                # dựng Postgres + Auth + Studio trong Docker
supabase db diff -f init      # tạo migration từ thay đổi
supabase link --project-ref <ref>
supabase db push              # đẩy migration lên cloud
```
Dùng migration (file SQL trong git) thay vì sửa tay trên dashboard.

## Lỗi thường gặp
- Dự án bị pause: vào dashboard bấm Restore.
- Truy vấn trả mảng rỗng: RLS đang chặn, kiểm tra policy và trạng thái đăng nhập.
- Lộ key: vào Settings → API để rotate key, rồi cập nhật lại app.
