# Khảo sát hiện trường — hướng dẫn triển khai

App khảo sát trạm biến áp / đường dây: nhập tọa độ GPS, chụp ảnh, lưu vào Firebase,
xem trên bản đồ, xuất CSV. Chạy độc lập trên điện thoại, không cần quay lại Claude.

## Bước 1 — Tạo project Firebase (miễn phí)

1. Vào https://console.firebase.google.com → **Add project** → đặt tên (VD: `khao-sat-qtpc`) → tạo xong.
2. Trong project, vào **Build > Firestore Database** → **Create database** → chọn chế độ
   **Production mode** → chọn khu vực gần nhất (VD: `asia-southeast1`).
3. Vào **Build > Authentication** → tab **Sign-in method** → bật **Google**. Khi được hỏi
   "Project support email", chọn email Google của anh → Save.
4. Vào **Project settings** (biểu tượng bánh răng) → mục **Your apps** → bấm icon `</>`
   (Web app) → đặt tên bất kỳ → **Register app**. Firebase sẽ hiện đoạn code chứa
   `apiKey`, `authDomain`, `projectId`... — copy các giá trị này.

## Bước 2 — Điền cấu hình vào app

Mở file `firebase-config.js`, dán các giá trị vừa copy vào:

```js
const FIREBASE_CONFIG = {
  apiKey: "AIza...",
  authDomain: "khao-sat-qtpc.firebaseapp.com",
  projectId: "khao-sat-qtpc",
  storageBucket: "khao-sat-qtpc.appspot.com",
  messagingSenderId: "...",
  appId: "..."
};
```

## Bước 3 — Đặt luật bảo mật Firestore

Trong Firebase Console → **Firestore Database** → tab **Rules**, dán (thay email bằng
đúng địa chỉ Gmail anh sẽ dùng để đăng nhập):

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /khao_sat/{docId} {
      allow read, write: if request.auth != null
        && request.auth.token.email == "thaidh24108@gmail.com";
    }
  }
}
```

Cách này chỉ cho phép đúng tài khoản Google của anh đọc/ghi dữ liệu — an toàn hơn hẳn
so với cho phép bất kỳ ai đăng nhập ẩn danh.

## Bước 4 — Đưa lên GitHub Pages

1. Tạo repo mới trên GitHub (có thể để **Private** nếu không muốn công khai code,
   GitHub Pages vẫn chạy được với repo private nếu tài khoản có GitHub Pro,
   hoặc để **Public** nếu không ngại — dữ liệu khảo sát không nằm trong code,
   nó nằm trong Firebase nên vẫn an toàn dù repo public).
2. Upload toàn bộ các file trong thư mục này (`index.html`, `firebase-config.js`,
   `manifest.json`, `sw.js`, `icon-192.png`, `icon-512.png`) lên repo.
3. Vào **Settings > Pages** của repo → **Source** chọn nhánh `main`, thư mục `/ (root)` → Save.
4. Sau 1-2 phút, GitHub cho địa chỉ dạng:
   `https://<tên-tài-khoản>.github.io/<tên-repo>/`

## Bước 5 — Cài lên màn hình chính điện thoại

1. Mở địa chỉ GitHub Pages ở trên bằng Chrome (Android) hoặc Safari (iPhone).
2. **Android (Chrome)**: menu ⋮ → **Thêm vào Màn hình chính** (Add to Home screen).
3. **iPhone (Safari)**: nút Chia sẻ → **Thêm vào MH chính** (Add to Home Screen).
4. Icon app sẽ xuất hiện như app thật, mở lên full màn hình, không thanh địa chỉ trình duyệt.

## Cấu trúc bản đồ (mới)

Từ bản này, app tổ chức dữ liệu theo **bản đồ** (dự án khảo sát), mỗi bản đồ chứa các
**điểm** khảo sát riêng — các bản đồ hoàn toàn độc lập với nhau.

- Sau khi đăng nhập, app hiện màn hình **"Chọn bản đồ khảo sát"** — tạo bản đồ mới
  (đặt tên, VD: "Tuyến 371 Vĩnh Linh") hoặc mở lại bản đồ đã có.
- Vào trong 1 bản đồ mới thấy 3 tab quen thuộc: Ghi nhận / Danh sách / Bản đồ — mọi thao
  tác (thêm điểm, nhập Excel/KMZ, xuất CSV/KMZ) chỉ áp dụng cho bản đồ đang mở.
- Bấm **"← Đổi bản đồ"** ở góc trên để quay lại màn hình chọn, mở bản đồ khác hoặc tạo mới.
- Xóa 1 bản đồ ở màn hình chọn sẽ xóa toàn bộ điểm bên trong, không thể hoàn tác.

## Ghi chú vận hành

- App dùng **Google Sign-In** — mở app lần đầu sẽ hiện màn hình đăng nhập, bấm
  "Đăng nhập bằng Google" và chọn đúng tài khoản Gmail của anh.
- **Quan trọng khi deploy lên GitHub Pages**: sau khi có địa chỉ GitHub Pages (bước 4),
  quay lại Firebase Console → Authentication → Settings → tab "Authorized domains" →
  bấm "Add domain" → dán domain GitHub Pages vào (dạng `<tên-tài-khoản>.github.io`,
  không cần phần đường dẫn sau đó). Nếu bỏ qua bước này, đăng nhập Google sẽ báo lỗi
  "domain not authorized" khi mở app từ GitHub Pages.
- Dữ liệu lưu trên Firestore, đồng bộ qua tài khoản Firebase — mở trên nhiều thiết bị
  vẫn thấy cùng dữ liệu.
- App có bật **offline persistence**: khảo sát ở vùng không sóng vẫn lưu được, khi có
  mạng lại sẽ tự đồng bộ lên Firebase.
- Gói miễn phí (Spark plan) của Firestore: 50.000 lượt đọc/ngày, 20.000 lượt ghi/ngày —
  dư sức cho một người dùng khảo sát hiện trường.
- Nếu sau này muốn nhiều đồng nghiệp cùng dùng, chỉ cần thêm tài khoản Auth (email/password)
  thay vì anonymous — hỏi lại tôi khi cần, sẽ chỉnh phần quyền truy cập.

## Bản đồ vệ tinh

Tab **Bản đồ** có 2 nút "Bản đồ" / "Vệ tinh" ở trên cùng để chuyển giữa nền OpenStreetMap
(dạng đường phố) và ảnh vệ tinh (Esri World Imagery, miễn phí, không cần API key) — dùng
ảnh vệ tinh để nhìn rõ mặt bằng thực tế khu vực vừa khảo sát.

## Nhập dữ liệu từ Excel

Tab **Danh sách > ↑ Nhập Excel**. File Excel cần có các cột (đặt tên đúng như dưới,
không phân biệt hoa/thường, thứ tự cột tùy ý):

| Mã | Loại | Loại cột | Vĩ độ | Kinh độ | Tình trạng | Ghi chú |
|---|---|---|---|---|---|---|
| TBA-014 | Trạm biến áp | | 16.7500 | 107.2000 | | |
| VT12 | Đường dây | Cột néo | 16.7510 | 107.2010 | | |

Chỉ 3 cột bắt buộc: **Mã**, **Vĩ độ**, **Kinh độ** — dòng nào thiếu sẽ bị bỏ qua và báo
số dòng bỏ qua sau khi nhập xong. Dùng cột này để đưa lưới tọa độ hiện trạng có sẵn
(từ Excel cũ) vào thẳng app, không cần đo lại.

## Nhập / mở file KMZ, KML

Tab **Danh sách > ↑ Nhập KMZ/KML** — hỗ trợ cả `.kmz` (từ Google Earth/Google My Maps
xuất ra) và `.kml`. App đọc các điểm (Placemark dạng Point) trong file, đưa vào danh sách
và hiển thị trên bản đồ với màu xám nhạt để phân biệt với điểm tự khảo sát.

Dùng tính năng này khi:
- Đi **bàn giao mặt bằng**: mở file KMZ thiết kế để đối chiếu ngay trên bản đồ vệ tinh
  tại hiện trường.
- Đi **thi công / giám sát**: mang theo file KMZ vị trí cột/trạm đã duyệt, so sánh với
  vị trí GPS thực tế đang đứng.

App đọc được cả 2 dạng: **điểm** (Point — cột, trạm) và **tuyến đường dây** (LineString —
đường dây vẽ sẵn từ CAD xuất KMZ). Tuyến đường dây hiển thị dạng đường nối liền trên bản đồ,
trong danh sách hiện số đỉnh và tổng chiều dài ước tính. Vùng khoanh (Polygon) chưa hỗ trợ,
sẽ báo số lượng bị bỏ qua sau khi nhập.

## Xuất KMZ

Tab **Danh sách > ↓ Xuất KMZ** — xuất toàn bộ điểm đã khảo sát thành file `.kmz`, mở
trực tiếp bằng Google Earth hoặc Google My Maps để trình bày, báo cáo, hoặc gửi cho
đơn vị thiết kế.
