# C# / .NET – nhiều SDK song song

Cập nhật lần cuối: 2026-11-14

.NET hỗ trợ cài **nhiều SDK cạnh nhau**. Dự án chọn SDK bằng file `global.json`. Không cần virtualenv: thư viện nằm trong NuGet cache và `obj/`, `bin/` của từng dự án.

Xem các bản đang được hỗ trợ: https://dotnet.microsoft.com/download/dotnet

## Cài

**Windows**
```powershell
winget search Microsoft.DotNet.SDK
winget install Microsoft.DotNet.SDK.8
winget install Microsoft.DotNet.SDK.9
```
**macOS**
```bash
brew install --cask dotnet-sdk          # bản mới nhất
# Bản cũ hơn: dùng installer chính thức hoặc script dotnet-install
curl -sSL https://dot.net/v1/dotnet-install.sh -o dotnet-install.sh
chmod +x dotnet-install.sh
./dotnet-install.sh --channel 8.0       # cài vào ~/.dotnet
export DOTNET_ROOT=$HOME/.dotnet
export PATH=$PATH:$DOTNET_ROOT
```

## Xem và chọn SDK
```bash
dotnet --list-sdks
dotnet --list-runtimes
dotnet --version        # SDK đang dùng tại thư mục hiện tại
```

## Pin SDK theo dự án
```bash
dotnet new globaljson --sdk-version 8.0.404 --roll-forward latestFeature
```
Kết quả `global.json`:
```json
{ "sdk": { "version": "8.0.404", "rollForward": "latestFeature" } }
```
`dotnet --version` trong thư mục có file này sẽ dùng đúng SDK 8.0.x, kể cả khi máy có bản mới hơn.

Chọn runtime đích trong `.csproj`:
```xml
<PropertyGroup><TargetFramework>net8.0</TargetFramework></PropertyGroup>
```

## Công cụ theo dự án (thay cho cài global)
```bash
dotnet new tool-manifest
dotnet tool install dotnet-ef       # ghi vào .config/dotnet-tools.json
dotnet tool restore                 # máy khác dựng lại
dotnet ef --version
```

## IDE
- Visual Studio (Windows), Rider (cả hai), VS Code + extension C# Dev Kit (cả hai).

## Lỗi thường gặp
- "A compatible .NET SDK was not found": `global.json` yêu cầu SDK chưa cài. Cài đúng bản hoặc nới `rollForward`.
- Chạy được trên máy này nhưng lỗi trên máy khác: thiếu runtime tương ứng `TargetFramework`.
