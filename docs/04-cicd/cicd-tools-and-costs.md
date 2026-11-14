# Công cụ CI/CD thông dụng và chi phí

Cập nhật lần cuối: 2026-11-14

> Giá và hạn mức đổi liên tục, các số dưới đây tổng hợp từ nhiều nguồn tại tháng 11/2026. Một số nguồn **mâu thuẫn nhau** (ghi chú ở từng mục). Luôn kiểm tra trang giá chính thức trước khi quyết định.
> Nhắc lại quy ước repo: không bật trigger tự động; chỉ chạy tay khi được yêu cầu.

## Bảng so sánh nhanh (gói free)

| Công cụ | Gói free (build trên máy của hãng) | Self-hosted runner | Hợp với |
|---|---|---|---|
| **GitHub Actions** | Repo public: không giới hạn. Repo private: 2.000 phút Linux/tháng, 500 MB storage | Có | Code đã ở GitHub |
| **GitLab CI/CD** | 400 phút/tháng, 5 user, 10 GiB | Có, không tính phút | Muốn nền tảng DevOps trọn bộ, hoặc tự host GitLab |
| **CircleCI** | Có gói free tính theo *credit* (xem ghi chú) | Có (gói trả phí) | Pipeline song song nhiều |
| **Bitbucket Pipelines** | 50 phút/tháng | — | Nhóm dùng Jira/Atlassian |
| **Azure DevOps** | 1 job song song, 1.800 phút/tháng, 5 user | 1 job self-hosted không giới hạn phút | Hệ sinh thái Microsoft, .NET |
| **Cloudflare Pages (build)** | 500 build/tháng, 1 build cùng lúc | Không | Web tĩnh/SSR lên Cloudflare |
| **Cloudflare Workers Builds** | 3.000 phút build/tháng | Không | Deploy Workers |
| **Bitrise** | Hobby: 300 credit/tháng, 1 app private | Không | CI chuyên mobile |
| **Expo EAS** | Số build mobile giới hạn/tháng, ưu tiên thấp | Build local miễn phí | App React Native/Expo ([mobile-release.md](../03-advanced/mobile-release.md)) |
| **Jenkins** | Phần mềm miễn phí (mã nguồn mở) | Bắt buộc tự host | Cần toàn quyền kiểm soát, on-prem |
| **Woodpecker / Drone** | Miễn phí, tự host | Tự host | Self-host nhẹ, cá nhân |

## Chi tiết chi phí từng nền tảng

### GitHub Actions
- Gói Free: 2.000 phút và 500 MB cho repo private; Pro 3.000 phút/1 GB; Team 3.000 phút/2 GB (Team 4 USD/user). Repo public chạy trên runner chuẩn là miễn phí.
- Đơn giá vượt hạn mức (sau cắt giảm 01/2026): Linux 0,006 USD/phút, Windows 0,010 USD/phút, macOS 0,062 USD/phút. Phút Windows tính gấp 2, macOS gấp 10 so với hạn mức miễn phí.
- **Lưu ý đã đổi nhiều lần**: GitHub công bố thu thêm 0,002 USD/phút cho runner self-hosted ở repo private từ 01/03/2026, nhưng có nguồn cho rằng việc này bị hoãn và self-hosted vẫn miễn phí tới 06/2026. Đọc mục *Billing* trong tài liệu GitHub để biết hiện trạng.
- Hết hạn mức mà chưa có phương thức thanh toán thì job bị chặn, không bị trừ tiền ngoài ý muốn. Phút không cộng dồn sang tháng sau.
- Mẹo tiết kiệm: dùng cache, `timeout-minutes`, tránh matrix thừa, tránh macOS nếu không cần.

### GitLab CI/CD
- Free: 400 phút/tháng (tính theo namespace cấp cao nhất), 5 user, 10 GiB. Premium 29 USD/user/tháng gồm 10.000 phút. Mua thêm 10 USD/1.000 phút. Runner tự host không tốn phút.
- Có chương trình open-source với hạn mức cao hơn cho dự án đủ điều kiện.

### CircleCI
- **Nguồn mâu thuẫn**: phần lớn nguồn ghi gói Free là 30.000 credit/tháng (khoảng 3.000 đến 6.000 phút tuỳ cỡ máy), nhưng một nguồn ghi 6.000 credit (~60 phút máy medium). Credit tiêu hao nhanh hơn với máy lớn. Gói Performance từ 15 USD/tháng. Hãy xem trang giá chính thức.

### Bitbucket Pipelines
- Free: 50 phút/tháng, 5 user. Bản trả phí tính theo user, phút thêm khoảng 10 USD/1.000 phút. Hạn mức free khá nhỏ.

### Azure DevOps
- Free: 1 job Microsoft-hosted song song với 1.800 phút/tháng, cộng 1 job self-hosted không giới hạn phút, 5 user.

### Cloudflare
- **Pages**: 500 build/tháng, mỗi build tối đa 20 phút, chạy một build cùng lúc, băng thông và request tĩnh không giới hạn. Pro (20 USD/tháng nếu trả theo năm) nâng lên 5.000 build.
- **Workers Builds**: gói free 3.000 phút/tháng, 1 build đồng thời; gói trả phí 6.000 phút rồi 0,005 USD/phút.
- Pages Functions/Workers tính theo hạn mức Workers: free 100.000 request/ngày.
- Deploy bằng `wrangler` từ máy mình **không dùng hạn mức build** của Cloudflare.

### Jenkins và các công cụ tự host
- Phần mềm miễn phí nhưng bạn trả tiền máy chủ và **công sức vận hành** (cập nhật, plugin, bảo mật). Một số ước tính hạ tầng cho Jenkins vào khoảng 1.200 đến 3.600 USD/năm ở quy mô nhóm. Điểm hoà vốn so với runner hosted được ước tính quanh vài nghìn phút build mỗi tháng (một nguồn nêu khoảng 8.000 phút/tháng/repo).
- "Self-hosted không miễn phí, chỉ là chuyển chi phí từ hoá đơn sang máy và con người."
- Với cá nhân, một VPS nhỏ hoặc máy cũ chạy Woodpecker/Drone/Jenkins là đủ, nhưng phải tự bảo mật (cổng mở, secret).

## Chọn thế nào

| Tình huống | Gợi ý |
|---|---|
| Một mình, repo private, build ít | **Không cần CI**: chạy `npm run check` và deploy bằng CLI từ máy. Cần thì dùng GitHub Actions `workflow_dispatch` (2.000 phút free) |
| Repo public / mã nguồn mở | GitHub Actions (miễn phí không giới hạn trên runner chuẩn) |
| Web tĩnh, deploy lên Cloudflare | `wrangler pages deploy` từ máy, hoặc Workers Builds (3.000 phút free) nếu muốn build trên cloud |
| App mobile cần build iOS mà không có Mac | EAS Build hoặc Bitrise (free có hạn mức nhỏ), tính trước chi phí phút macOS |
| Công ty, dữ liệu không được ra ngoài | GitLab self-managed hoặc Jenkins trên hạ tầng nội bộ |
| Nhóm dùng Atlassian | Bitbucket Pipelines (hạn mức free rất nhỏ, thường phải trả phí) |

## Ước tính chi phí nhanh
Công thức: `phút/lần × số lần/tháng × hệ số OS`. Ví dụ job Linux 8 phút, chạy tay 60 lần/tháng = 480 phút, nằm trong 2.000 phút free của GitHub. Cùng job trên macOS = 480 × 10 = 4.800 phút quy đổi, vượt hạn mức; phần vượt tính theo 0,062 USD/phút.

Cũng cần tính chi phí **không phải phút build**: storage artifact/cache, băng thông, người vận hành (nếu self-host), tài khoản store cho mobile.

## Cảnh báo chi phí
- Đặt giới hạn chi tiêu (spending limit) về 0 USD ở GitHub/GitLab nếu muốn chắc không bị tính tiền.
- Không lưu thẻ ở nền tảng nếu chỉ dùng gói free và muốn tự động chặn khi hết hạn mức.
- Đặt `timeout-minutes` cho mọi job.

## Nguồn tham khảo (kiểm tra lại giá)
- GitHub Actions billing: https://docs.github.com/billing/managing-billing-for-your-products/managing-billing-for-github-actions
- GitHub Actions changelog 12/2025: https://github.blog/changelog/2025-12-16-coming-soon-simpler-pricing-and-a-better-experience-for-github-actions/
- GitLab pricing: https://about.gitlab.com/pricing
- CircleCI pricing: https://circleci.com/pricing
- Bitbucket pricing: https://www.atlassian.com/software/bitbucket/pricing
- Cloudflare Pages limits: https://developers.cloudflare.com/pages/platform/limits
- Cloudflare Workers Builds: https://developers.cloudflare.com/workers/ci-cd/builds/limits-and-pricing/
- Expo pricing: https://expo.dev/pricing
