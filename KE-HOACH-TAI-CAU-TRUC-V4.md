# KẾ HOẠCH TÁI CẤU TRÚC TOÀN DIỆN (MỨC 2) — PHẦN MỀM KẾ TOÁN, HÓA ĐƠN & THUẾ CHUYÊN NGHIỆP (V4)

*Ngày lập: 08/11/2026 · Nền tảng xuất phát: XML-Pro v3.0.8 (`c3dc342`)*
*Kho mã nguồn mới: `F:\AI\9.ketoan-app` · GitHub: https://github.com/tuongotpho/ketoan-app.git*
*Mục tiêu: Chuyển đổi triệt để từ giao diện "Trình đọc tệp XML lẻ đa ngành" sang **Phần mềm Quản lý Hóa đơn – Kế toán – Thuế Đa doanh nghiệp (Kế Toán Pro V4)** chuẩn Desktop chuyên nghiệp, tốc độ tức thì, 100% xử lý cục bộ trên máy.*

---

## 1. Định vị Sản phẩm & 5 Nguyên tắc Kiến trúc Bất biến

1. **Định vị thuần Kế toán – Hóa đơn – Thuế:**
   - Lấy **Doanh nghiệp / Hộ kinh doanh (`MST`) + Kỳ tính thuế (`Năm / Quý / Tháng`)** làm trung tâm điều hướng thay vì lấy "danh sách tệp đang kéo thả" làm trung tâm.
   - Phục vụ tối ưu cả **Kế toán nội bộ 1 công ty** lẫn **Đại lý thuế / Kế toán dịch vụ quản lý 10–50 MST** trên cùng một máy.
   - Giữ ngầm bộ giải mã XML phụ trợ (`nganh-doc.js`) để không lỗi khi người dùng mở tệp lạ ở phân hệ đọc XML, nhưng **gỡ bỏ hoàn toàn thông tin giới thiệu 9 ngành (Y tế BHYT, Đấu thầu, Hải quan…)** khỏi màn hình chính và điều hướng.
2. **An toàn & Ngoại tuyến 100% (Offline-First):**
   - Giữ nguyên chính sách `Content-Security-Policy` khóa mạng của trang (`connect-src 'self' https://tep.xml-pro.example`), vỏ Windows `WebView2` (`VoWebView2.cs`) biên dịch bằng `csc.exe` có sẵn trên Windows.
   - Chỉ kết nối trực tiếp Cổng Thuế (`hoadondientu.gdt.gov.vn`) khi người dùng chủ động mở khung tải hóa đơn (không lưu mật khẩu, không qua máy chủ trung gian).
3. **Bảo toàn 100% Lõi Nghiệp vụ đã kiểm chứng (`45/45` test):**
   - Giữ nguyên độ chính xác của các mô-đun cốt lõi: kiểm tra chữ ký số XMLDSig (`verifySig` khớp `.NET SignedXml`), dựng tờ khai XSLT `iTax Viewer` (`mau-itax.js`), đối chiếu tờ khai `01/GTGT` (`doi-chieu.js`), lập tờ khai `01/GTGT` / `04/GTGT` / `01/CNKD` / `KHBS` (`lap-to-khai.js`), tính thuế TNDN Luật 67/2025/QH15 (`dashboard-ke-toan.js`), kiểm toán 4 nhóm rủi ro HĐ & xuất MISA tự học định khoản (`kiem-toan-misa.js`), khóa sổ & ghi chú (`so-tay.js`).
4. **Chuẩn Thẩm mỹ Phần mềm Kế toán Chuyên nghiệp:**
   - **Mặc định Giao diện Sáng (Light Mode)** nền đặc sắc nét (`#ffffff` / `#f8fafc`, không nền mờ trong suốt `backdrop-filter`), hỗ trợ Chế độ Tối tương phản $\ge 3:1$.
   - **Không Emoji (`0 emoji`) trên toàn bộ ứng dụng:** thay toàn bộ biểu tượng cảm xúc ở thanh bên, nút bấm, hộp thoại bằng hệ thống **SVG Icon nét mảnh đồng bộ (`16px` / `18px`)**.
   - **Mật độ hiển thị chuẩn bảng tính:** dòng bảng gọn `28–30px`, phông số `font-variant-numeric: tabular-nums` thẳng hàng chữ số tuyệt đối, hộp thoại bảng kê rộng toàn khung (`96vw`).
5. **Đóng gói gọn nhẹ không phụ thuộc Node Bundler:**
   - Tách mô-đun `.css` / `.js` sạch sẽ nhưng nạp trực tiếp không cần Webpack/Vite để giữ quy trình biên dịch 1 lệnh bằng PowerShell + `csc.exe` (`dong-goi\dong-goi.ps1`).

---

## 2. Kiến trúc Khung Giao diện Mới (App Shell Mức 2)

### 2.1. Sơ đồ bố cục tổng thể

```text
+--------------------------------------------------------------------------------------------------+
| [Logo] KẾ TOÁN PRO [Công ty: 2601102955 - TNHH TM NGA ANH PHÚ THỌ v] [Kỳ: Năm 2026 | Quý 4 v]    |
|                    [+ Nạp XML/ZIP] [Tải Cổng Thuế] [Xuất MISA/Excel v] [Tìm Ctrl+K] [Sáng/Tối]   |
+--------------------+---------------------------------------------+-------------------------------+
| ĐIỀU HƯỚNG (210px) | KHUNG NGHIỆP VỤ CHÍNH (Co giãn 100%)        | NGĂN XEM NHANH (500px, ẩn/mở) |
| (Thu gọn 52px      |                                             |                               |
|  bằng Ctrl+B)      |  - Thẻ chỉ số KPI chuẩn xác                 |  [Bản thể hiện Hóa đơn A4]    |
|                    |  - Bảng dữ liệu mật độ cao (28px/dòng)      |  - Trạng thái Chữ ký số       |
| 1. Danh bạ Công ty |  - Bộ lọc nhanh theo trạng thái / rủi ro    |  - Cảnh báo Kiểm toán         |
| 2. Tổng quan Thuế  |                                             |  - Ô Ghi chú & Đánh dấu       |
| 3. Sổ Hóa đơn      |                                             |  - Phím [↑] [↓] duyệt liên    |
| 4. Kiểm toán Rủi ro|                                             |    tiếp không mất bảng lọc    |
| 5. Tờ khai & Đ.Chiếu                                             |                               |
| 6. Xuất MISA & BK  |                                             |                               |
| 7. Đọc XML & CKS   |                                             |                               |
+--------------------+---------------------------------------------+-------------------------------+
```

### 2.2. Chi tiết 3 thành phần đột phá của App Shell

1. **Thanh Ngữ cảnh Toàn cục (`Global Topbar` — cao `48px`, cố định trên cùng):**
   - **Bộ chọn Công ty toàn cục (`#ctxMst`):** Hiển thị `MST — Tên doanh nghiệp / Hộ KD`. Khi chuyển công ty tại đây, toàn bộ các phân hệ (*Tổng quan Thuế, Sổ Hóa đơn, Kiểm toán, Tờ khai, Xuất MISA*) tự động chuyển dữ liệu sang đúng công ty đó (xóa bỏ tình trạng 4 ô chọn MST rời rạc như bản v3).
   - **Bộ chọn Kỳ làm việc toàn cục (`#ctxNam`, `#ctxKy`):** Chọn nhanh `Cả năm` / `Quý 1..4` / `Tháng 01..12`.
   - **Cụm nút Hành động nhanh:** `+ Nạp XML / Thư mục` (thay cho khung nét đứt chiếm chỗ cũ), `Tải HĐ Cổng Thuế`, `Xuất nhanh ▾` (Bảng kê Excel tổng hợp / Xuất MISA / Đổi tên file hàng loạt), `Tìm kiếm toàn cục (Ctrl+K)`.

2. **Thanh Điều hướng 7 Phân hệ (`Sidebar Rail` — đóng/mở `210px ↔ 52px` bằng `Ctrl+B`):**
   - **Phân hệ 1 — `Danh bạ Công ty` (Workspace Launcher):** Thay thế hoàn toàn màn hình chào `#homeEmpty` cũ. Hiển thị lưới thẻ các Doanh nghiệp / Hộ KD đang quản lý trên máy: MST, tên đơn vị, tổng số HĐ Mua/Bán, trạng thái tờ khai kỳ gần nhất, số cảnh báo rủi ro tồn đọng, nút bấm 1 chạm vào làm việc ngay hoặc tải bù hóa đơn Cổng Thuế.
   - **Phân hệ 2 — `Tổng quan Thuế & Lợi nhuận`:** Kế thừa và tối ưu không gian rộng của Dashboard 2 mảng cốt lõi:
     - *Thuế GTGT:* Sổ kê khai theo kỳ (gập/mở năm), trạng thái Khóa sổ, Đối chiếu tờ khai, Lập tờ khai nhanh.
     - *Thuế TNDN & Lợi nhuận sau thuế:* Bảng tính KQKD Quý 1..4 & Cả năm (15%/17%/20% Luật 67/2025/QH15), bù lỗ tự động, điều chỉnh `[B4]` / `[C3]`.
   - **Phân hệ 3 — `Sổ Hóa đơn Mua / Bán`:** Hợp nhất `Kho hóa đơn` và bảng chi tiết hóa đơn thành một sổ tra cứu duy nhất tốc độ cao ($\ge 50.000$ HĐ): lọc Mua vào / Bán ra / Máy tính tiền / Hóa đơn thay thế – điều chỉnh / Đã hạch toán / Đã thanh toán; thống kê Top Khách hàng & Nhà cung cấp.
   - **Phân hệ 4 — `Kiểm toán Rủi ro Thuế`:** Màn hình chuyên biệt dành cho kế toán trưởng rà soát trước kỳ quyết toán/thanh tra:
     - Rủi ro Ký lệch ngày/vắt kỳ, Tiền mặt $\ge 5$tr & cộng dồn cùng ngày, Thủng dải số bán ra, Sai tên/ĐC/MST người mua, Xuất khống / Âm kho chứng từ & Bán dưới giá vốn, Đối soát Sao kê Ngân hàng & Cổng Thuế.
   - **Phân hệ 5 — `Tờ khai & Đối chiếu HTKK`:** Quản lý tập trung toàn bộ tệp tờ khai (`01/GTGT`, `04/GTGT`, `01/CNKD`, `05/KK-TNCN`, `03/TNDN`, `BCTC`, Giấy nộp tiền) của công ty đang chọn; hiển thị kết quả đối chiếu từng chỉ tiêu, Kiểm chéo 12 chỉ tiêu BCTC, Đối chiếu Lương – TNCN – BHXH, và nút `Lập tờ khai mới` / `Lập KHBS` / `Nháp 03/TNDN`.
   - **Phân hệ 6 — `Kết xuất MISA & Bảng kê`:** Đưa giao diện Định khoản thông minh tự học (`TT200/TT99` & `TT133`) thành một màn hình làm việc toàn trang thoáng rộng (không cần gò trong popup), kèm xuất Bảng kê `01-1/GTGT`, `01-2/GTGT`, `MISA_Mua_hang`, `MISA_Ban_hang`.
   - **Phân hệ 7 — `Đọc XML & Chữ ký số`:** Giữ nguyên trải nghiệm mở nhiều tab chứng từ, xem biểu mẫu chuẩn XSLT `iTax Viewer`, in PDF từng biểu/tất cả biểu, kiểm tra chi tiết chứng thư số X.509 và cây thư mục Explorer.

3. **Ngăn Xem nhanh Hóa đơn cạnh Bảng kê (`Split Preview Drawer`):**
   - Ở `Sổ Hóa đơn` và `Kiểm toán Rủi ro`, bấm 1 dòng hóa đơn sẽ mở ngăn phải (`45%` chiều ngang hoặc `520px`) hiển thị ngay bản thể hiện Hóa đơn + Chữ ký số + Ô ghi chú.
   - Hỗ trợ phím mũi tên **`↑` / `↓`** để lướt kiểm tra hàng chục hóa đơn liên tiếp trong tích tắc mà mắt vẫn nhìn thấy dòng đang chọn trên bảng kê; bấm `Esc` để đóng ngăn xem nhanh, bấm đúp để mở sang tab đầy đủ.

---

## 3. Hợp nhất Kiến trúc Lưu trữ Dữ liệu (`XmlProKho v2`)

| Object Store | Khóa chính (`keyPath`) | Nội dung lưu trữ | Mục đích & Tối ưu tốc độ |
|---|---|---|---|
| **`cong_ty`** | `mst` | `mst`, `ten`, `dchi`, `cheDoKt` (`TT200`/`TT133`/`HKD`), `loaiKy` (`thang`/`quy`), `soHdMua`, `soHdBan`, `capNhatLuc`, `ghim` | Dựng tức thì màn hình **Danh bạ Công ty** và thanh chọn Công ty toàn cục khi vừa mở app (`< 50ms`). |
| **`hoadon`** | `k` (`chu\|mstBan\|mau\|kh\|so`) | Giữ tương thích ngược v3 + bổ sung: `httt`, `ngayKy`, `dchiBan`, `dchiMua`, `tsChinh`, `soDongHang`, `coXmlGoc`, `ruiRo` | Chỉ mục siêu nhẹ trên RAM: lọc, cộng tổng GTGT/TNDN và chạy Kiểm toán cho `50.000` hóa đơn dưới `1,2 giây`. |
| **`xml_goc`** | `k` | `k`, `xml` (chuỗi XML gốc), `tenTep`, `duongDan`, `kyBan`, `kyCqt` | Tách riêng chuỗi XML nặng khỏi bảng chỉ mục `hoadon`. Khôi phục đầy đủ dòng hàng, chữ ký và kiểm toán khi mở lại app. |
| **`to_khai`** | `id` (`mst\|maTKhai\|ky\|lan`) | `mst`, `kind` (`tk`/`tb`/`ct`), `maTKhai`, `title`, `ky`, `ngayNop`, `xml`, `tenTep`, `duongDan` | Lưu bền vững Tờ khai thuế, BCTC và Giấy nộp tiền theo từng công ty. |

---

## 4. Lộ trình Thi công 5 Giai đoạn Toàn diện

1. **Giai đoạn 1:** Dựng App Shell Mức 2 (`giao-dien.css`, `bieu-tuong.js`, `khung-ung-dung.js`), Global Topbar, Sidebar 7 phân hệ (`Ctrl+B`), Tìm kiếm toàn cục (`Ctrl+K`), xóa sạch Emoji (`0 emoji`), Light/Dark chuẩn tương phản $\ge 3:1$.
2. **Giai đoạn 2:** Hợp nhất Kho Đa công ty `XmlProKho v2` (`kho-du-lieu.js`) + Phân hệ 1 `Danh bạ Công ty` (`ban-lam-viec.js`) — mở lại ứng dụng có ngay toàn bộ dữ liệu đa công ty không cần nạp lại thư mục XML.
3. **Giai đoạn 3:** Sổ Hóa đơn Hợp nhất (Phân hệ 3) + Ngăn Xem nhanh `Split Preview Drawer` (`↑/↓`, `Esc`) + Tách trang độc lập cho Phân hệ 4 (`Kiểm toán Rủi ro`) & Phân hệ 6 (`Kết xuất MISA & Bảng kê`).
4. **Giai đoạn 4:** Nâng cấp Nghiệp vụ Kế toán Chuyên sâu (`nghiep-vu-chuyen-sau.js`):
   - **Nhóm 3:** Khớp Sao kê Ngân hàng (`camt.052/053` / Excel / CSV) ↔ Hóa đơn & Quản trị Công nợ/Tuổi nợ `131`/`331`, chứng minh thanh toán không dùng tiền mặt $\ge 5$ triệu đ.
   - **Nhóm 5:** Kiểm soát Giá vốn, phát hiện **Xuất khống / Âm kho chứng từ** (xuất bán trước ngày mua vào đầu tiên) & cảnh báo bán dưới giá vốn bình quân.
   - **Nhóm 4 & Nhóm 6:** Đối chiếu chéo Lương – TNCN (`05/KK`, `05/QTT`) – BHXH (`D02-LT`) + Kiểm chéo **12 chỉ tiêu sống còn trên BCTC** + Sinh nháp Tờ khai Quyết toán **`03/TNDN`** XML chuẩn HTKK.
5. **Giai đoạn 5:** Đồng bộ nhận diện thương hiệu V4, cập nhật bộ đóng gói `.exe` / `.msix` (`dong-goi.ps1`, `CaiDat.cs`, `VoWebView2.cs`, `Chung.cs`), chạy kiểm thử tự động đạt 100% PASS và đẩy lên GitHub `ketoan-app`.
