# Firebase

Cập nhật lần cuối: 2026-11-14

## Dùng để làm gì
Hosting web tĩnh/SPA, **Authentication**, **Firestore** (NoSQL), Cloud Storage, **FCM** (push notification), Analytics, Crashlytics. Mạnh cho app mobile (Flutter, React Native, Android, iOS).

## Gói free
Gói **Spark** miễn phí có hạn mức hàng ngày/tháng. Một số tính năng (Cloud Functions, một số Storage) yêu cầu gói **Blaze** (trả theo mức dùng, cần thẻ, nhưng vẫn có hạn mức free). Xem: https://firebase.google.com/pricing. Nên đặt **budget alert** trên Google Cloud khi dùng Blaze.

## Setup
1. Vào https://console.firebase.google.com, đăng nhập Google, **Add project**.
2. Trong project: **Add app** (Web, Android, iOS) để lấy cấu hình.
3. Bật dịch vụ cần dùng: **Build → Authentication / Firestore Database / Storage**.

## Cài Firebase CLI
```bash
npm install -g firebase-tools      # cần Node, xem ../01-environment/node.md
firebase --version
firebase login
```

## Deploy web lên Firebase Hosting
```bash
cd myapp
firebase init hosting            # chọn project, thư mục build (dist/build), SPA = yes
# Khi được hỏi "Set up automatic builds with GitHub?" → trả lời No (không bật CI tự động)
npm run build
firebase deploy --only hosting
```
Kết quả: URL dạng `https://<project-id>.web.app`. Gắn domain riêng ở **Hosting → Add custom domain**.

## Dùng trong app web
```bash
npm install firebase
```
```js
import { initializeApp } from 'firebase/app'
import { getAuth, GoogleAuthProvider, signInWithPopup } from 'firebase/auth'
const app = initializeApp({ apiKey: '...', authDomain: '...', projectId: '...', appId: '...' })
await signInWithPopup(getAuth(app), new GoogleAuthProvider())
```
Firebase `apiKey` phía web không phải secret, nhưng **Security Rules của Firestore/Storage bắt buộc phải viết**.

## Firestore Security Rules mẫu
```
rules_version = '2';
service cloud.firestore {
  match /databases/{db}/documents {
    match /users/{uid}/{doc=**} {
      allow read, write: if request.auth != null && request.auth.uid == uid;
    }
  }
}
```

## Chọn Firebase hay Supabase
- Cần SQL, quan hệ giữa bảng, muốn tránh khoá nhà cung cấp: Supabase.
- App mobile, push notification, Crashlytics, realtime đơn giản: Firebase.
- Có thể dùng kết hợp (ví dụ FCM của Firebase + DB của Supabase).
