# Docker

Cập nhật lần cuối: 2026-11-14

## Dùng để làm gì
- Chạy DB/service local (Postgres, Redis...) mà không cài vào máy.
- Đóng gói app thành image để deploy ở bất kỳ đâu.
- Nền tảng cho [Dev Container](../01-environment/devcontainer.md).

## Gói free
Docker Desktop miễn phí cho cá nhân, học tập, dự án nguồn mở và doanh nghiệp nhỏ; công ty lớn cần gói trả phí. Kiểm tra điều kiện hiện hành: https://www.docker.com/pricing. Lựa chọn thay thế: Rancher Desktop, Colima (macOS), Podman.

## Cài đặt

**Windows**
```powershell
wsl --install                       # bật WSL2, khởi động lại máy
winget install Docker.DockerDesktop
docker --version
docker run --rm hello-world
```
**macOS**
```bash
brew install --cask docker          # mở Docker.app lần đầu để cấp quyền
docker --version
docker run --rm hello-world
```
Apple Silicon: image `linux/amd64` chạy qua giả lập; ưu tiên image hỗ trợ `arm64`.

## Lệnh cơ bản
```bash
docker ps -a                        # container
docker images                       # image
docker logs -f <container>
docker exec -it <container> sh
docker system prune                 # dọn rác (cẩn thận)
```

## Ví dụ: Postgres local bằng Compose
`compose.yaml`
```yaml
services:
  db:
    image: postgres:16
    environment:
      POSTGRES_USER: app
      POSTGRES_PASSWORD: app_local_only
      POSTGRES_DB: appdb
    ports: ["5432:5432"]
    volumes: [pgdata:/var/lib/postgresql/data]
volumes:
  pgdata:
```
```bash
docker compose up -d
docker compose down            # giữ dữ liệu
docker compose down -v         # xoá luôn dữ liệu
```

## Ví dụ Dockerfile cho app Node
```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --omit=dev
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
```
Kèm `.dockerignore`: `node_modules`, `.git`, `.env`.

## Lỗi thường gặp
- Windows: "WSL 2 installation is incomplete": chạy `wsl --update`, bật Virtualization trong BIOS.
- Cổng bị chiếm: đổi `"5433:5432"`.
- Không đưa secret thật vào image hoặc compose commit lên git.
