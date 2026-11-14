# Đóng gói và phát hành app mobile (Expo / React Native)

Cập nhật lần cuối: 2026-11-14

## Chi phí tài khoản cửa hàng (không có đường free hoàn toàn)
| Cửa hàng | Phí |
|---|---|
| Google Play Console | **25 USD một lần**, không gia hạn hằng năm |
| Apple Developer Program | **99 USD/năm** |

Cả hai có thể yêu cầu xác minh danh tính bằng giấy tờ. Một tài khoản đăng được nhiều app. Kiểm tra mức phí hiện hành tại https://developer.apple.com/programs và trợ giúp Google Play Console.

Muốn thử không tốn phí: chạy trên máy thật qua **Expo Go**, hoặc cài file **APK** trực tiếp (Android). iOS bản thật bắt buộc có tài khoản Apple Developer.

## Các cách phân phối
| Cách | iOS | Android | Ghi chú |
|---|---|---|---|
| Expo Go | Có | Có | Chỉ để dev, hạn chế native module tuỳ biến |
| Development build | Cần tài khoản Apple | Có | Có native module, dùng khi dev |
| APK/AAB trực tiếp | Không | Có | Gửi file cho người dùng thử |
| TestFlight | Có | Không | Cần tài khoản Apple Developer |
| Google Play (Internal/Closed testing) | Không | Có | Thử nghiệm trước khi công khai |
| Store công khai | Có | Có | Qua review |
| PWA (web) | Cài từ trình duyệt | Cài từ trình duyệt | Không qua store, giới hạn một số tính năng |

## Build bằng EAS (Expo Application Services)
```bash
npm install -g eas-cli
eas login
eas build:configure
eas build --platform android --profile preview     # APK để thử
eas build --platform android --profile production  # AAB để lên Play
eas build --platform ios --profile production
```
**Gói free**: có số lượng build giới hạn mỗi tháng, ưu tiên thấp (hàng đợi có thể chờ lâu), và giới hạn thời gian mỗi build. Số liệu cụ thể thay đổi giữa các nguồn, xem https://expo.dev/pricing. Build lỗi rất nhanh thường không bị tính vào hạn mức.

### Build local (không tốn hạn mức)
```bash
eas build --platform android --local
```
- Android: cần JDK ([java.md](../01-environment/java.md)) và Android SDK, chạy được trên Windows và macOS.
- **iOS: chỉ build local được trên macOS** (cần Xcode).

> Theo quy ước của repo này, build chạy tay khi cần. Không bật tự động build mỗi lần push trên GitHub hay Expo.

## Cập nhật không cần qua store: EAS Update
Gửi bản cập nhật JavaScript/asset trực tiếp tới app đã cài (không áp dụng khi đổi code native). Gói free có giới hạn số người dùng active hằng tháng và băng thông, xem trang giá.
```bash
eas update --branch production --message "fix: sửa lỗi nhập liệu"
```

## Biến môi trường và secret
- Biến có tiền tố `EXPO_PUBLIC_` được **đóng gói vào app**, ai cũng đọc được. Không để secret ở đó.
- Secret của build: `eas secret:create` (hoặc `eas env:create` theo CLI hiện hành, kiểm tra `eas --help`).
- Xem thêm [secrets-management.md](secrets-management.md).

## Đăng nhập Google trên mobile
Cần OAuth Client ID loại Android (package name + SHA-1) và iOS (Bundle ID). SHA-1 của bản release khác bản debug. Xem [google-oauth.md](../02-services/google-oauth.md).

## Yêu cầu trước khi lên store
- Chính sách quyền riêng tư (URL công khai), mô tả, ảnh chụp màn hình, icon.
- Khai báo dữ liệu thu thập (Data safety của Google, Privacy labels của Apple).
- Google Play với tài khoản cá nhân mới có thể yêu cầu giai đoạn **closed testing** với một số người thử trước khi được công khai. Đọc yêu cầu hiện hành trong Play Console.
- Tăng `versionCode` (Android) / `buildNumber` (iOS) mỗi lần nộp.
- Apple review thường mất từ vài giờ đến vài ngày; có thể bị từ chối lần đầu, chuẩn bị tài khoản demo cho reviewer.

## Lỗi thường gặp
- Build iOS báo thiếu credentials: chạy `eas credentials` để tạo/chọn certificate và provisioning profile.
- Google sign-in lỗi trên bản release: chưa thêm SHA-1 của keystore release (hoặc của Play App Signing) vào Client ID.
- App crash chỉ ở bản release: xem log bằng `adb logcat` (Android) hoặc Xcode Organizer (iOS).
