# Dev Container – cô lập tuyệt đối theo dự án

Cập nhật lần cuối: 2026-11-14

Dev Container cho mỗi dự án một "máy ảo nhẹ" bằng Docker, gồm cả ngôn ngữ, công cụ và extension. Máy Windows và macOS dùng cùng một môi trường, hết lỗi "máy tôi chạy được".

## Điều kiện
- Docker Desktop (xem [docker.md](../02-services/docker.md)).
- VS Code + extension **Dev Containers**.

## Ví dụ `.devcontainer/devcontainer.json`
```json
{
  "name": "myapp",
  "image": "mcr.microsoft.com/devcontainers/base:ubuntu",
  "features": {
    "ghcr.io/devcontainers/features/node:1": { "version": "20" },
    "ghcr.io/devcontainers/features/python:1": { "version": "3.12" },
    "ghcr.io/devcontainers/features/java:1": { "version": "21" },
    "ghcr.io/devcontainers/features/dotnet:2": { "version": "8.0" }
  },
  "forwardPorts": [3000, 8000],
  "customizations": { "vscode": { "extensions": ["ms-python.python", "dbaeumer.vscode-eslint"] } }
}
```
Chỉ giữ các feature dự án cần. Mở thư mục trong VS Code, chọn **Reopen in Container**.

## Khi nào dùng
- Dự án cần nhiều ngôn ngữ cùng lúc, hoặc thư viện hệ thống đặc thù (C++).
- Nhiều người cùng làm, muốn môi trường giống hệt nhau.

## Hạn chế
- Tốn RAM/đĩa. Trên Windows nên để mã nguồn trong WSL2 để I/O nhanh.
- Không thay thế việc pin version bằng `.node-version`, `global.json`... Hai thứ nên đi cùng nhau.
