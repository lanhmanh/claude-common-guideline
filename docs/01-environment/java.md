# Java – đổi version và môi trường riêng từng dự án

Cập nhật lần cuối: 2026-11-14

Java không có "virtualenv". Cách cô lập là: **cài nhiều JDK song song** và **chỉ định JDK theo dự án** (qua `JAVA_HOME` hoặc toolchain của build tool).

Các bản LTS thường dùng: 8, 11, 17, 21, 25. Kiểm tra bản LTS mới nhất tại https://adoptium.net.

## macOS

### Cách 1: SDKMAN (khuyên dùng, đổi version theo thư mục)
```bash
curl -s "https://get.sdkman.io" | bash
source "$HOME/.sdkman/bin/sdkman-init.sh"

sdk list java                 # xem các bản có sẵn
sdk install java 21.0.5-tem   # ví dụ: Temurin 21 (đổi mã theo danh sách)
sdk install java 17.0.13-tem
sdk use java 17.0.13-tem      # chỉ trong terminal hiện tại
sdk default java 21.0.5-tem   # mặc định toàn máy
```
Pin theo dự án: trong thư mục dự án chạy `sdk env init` để tạo `.sdkmanrc`, sau đó `sdk env` để áp dụng (bật `sdkman_auto_env=true` trong `~/.sdkman/etc/config` để tự áp dụng khi `cd`).

### Cách 2: Homebrew + java_home
```bash
brew install --cask temurin@17 temurin@21
/usr/libexec/java_home -V                       # liệt kê JDK
export JAVA_HOME=$(/usr/libexec/java_home -v 17)
echo $JAVA_HOME && java -version
```
Thêm alias vào `~/.zshrc`:
```bash
alias j17='export JAVA_HOME=$(/usr/libexec/java_home -v 17)'
alias j21='export JAVA_HOME=$(/usr/libexec/java_home -v 21)'
```

## Windows (PowerShell)

### Cách 1: Scoop
```powershell
scoop bucket add java
scoop install temurin17-jdk temurin21-jdk
scoop reset temurin17-jdk      # đặt 17 làm bản đang dùng
java -version
scoop reset temurin21-jdk      # đổi sang 21
```

### Cách 2: winget + đặt JAVA_HOME theo phiên
```powershell
winget search Temurin
winget install EclipseAdoptium.Temurin.17.JDK
winget install EclipseAdoptium.Temurin.21.JDK

# Đổi version chỉ trong cửa sổ PowerShell hiện tại
$env:JAVA_HOME = "C:\Program Files\Eclipse Adoptium\jdk-17.0.13.11-hotspot"
$env:Path = "$env:JAVA_HOME\bin;$env:Path"
java -version
```
(Đường dẫn thật phụ thuộc bản đã cài: xem trong `C:\Program Files\Eclipse Adoptium`.)

Mẹo: tạo file `use-jdk17.ps1` trong thư mục dự án chứa 2 dòng đặt `JAVA_HOME` và `Path` ở trên, chạy `. .\use-jdk17.ps1` mỗi lần mở terminal.

> SDKMAN chỉ chạy trên Windows qua WSL hoặc Git Bash, không chạy native PowerShell.

## Chỉ định JDK ngay trong build tool (độc lập máy)

**Gradle (toolchain)**
```kotlin
// build.gradle.kts
java { toolchain { languageVersion.set(JavaLanguageVersion.of(21)) } }
```
**Maven**: dùng `maven.compiler.release` hoặc `maven-toolchains-plugin`; luôn dùng wrapper:
```bash
./mvnw -v      # macOS
.\mvnw.cmd -v  # Windows
```

## Lỗi thường gặp
- `java -version` ra bản cũ dù đã cài bản mới: `JAVA_HOME`/`Path` đang trỏ bản khác. Xem `where java` (Windows) / `which -a java` (macOS).
- IDE (IntelliJ/VS Code) dùng JDK khác terminal: đặt JDK trong Project Structure / `java.configuration.runtimes`.
