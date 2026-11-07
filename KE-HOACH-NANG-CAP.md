# KẾ HOẠCH NÂNG CẤP VÀ LỘ TRÌNH PHÁT TRIỂN — KẾ TOÁN PRO V4

*Cập nhật lần cuối: 08/11/2026 · Kho mã nguồn: `F:\AI\9.ketoan-app` (`https://github.com/tuongotpho/ketoan-app.git`)*

> **Bản thiết kế tái cấu trúc toàn diện Mức 2 — V4:** xem chi tiết tại [KE-HOACH-TAI-CAU-TRUC-V4.md](KE-HOACH-TAI-CAU-TRUC-V4.md).

Tài liệu này tổng hợp nền tảng tính năng kế thừa từ v3.0.8 và chi tiết các nhóm nghiệp vụ chuyên sâu Kế toán – Hóa đơn – Thuế (Nhóm 1 → Nhóm 6, 100% xử lý nội bộ trên máy người dùng).

---

## Phần A. Nền tảng Cốt lõi Kế thừa & Bảo toàn 100%

1. **Vỏ ứng dụng Windows `WebView2` (`VoWebView2.cs`):** Biên dịch bằng `csc.exe` có sẵn trên Windows, khóa mạng trang chính bằng CSP (`connect-src 'self' https://tep.xml-pro.example`), đóng gói `.exe` và `.msix`.
2. **Xác thực chữ ký số XMLDSig ngoại tuyến (`Web Crypto`):** Chuẩn hóa C14N và kiểm tra chữ ký RSA/ECDSA từ chứng thư X.509 đính kèm, khớp 100% `.NET SignedXml`.
3. **Hiển thị Tờ khai / BCTC chuẩn `iTax Viewer` (`mau-itax.js`):** Hỗ trợ đầy đủ các mẫu tờ khai HTKK/eTax và Báo cáo tài chính (TT200, TT133, TT99).
4. **Đối chiếu & Lập tờ khai thuế (`doi-chieu.js`, `lap-to-khai.js`):** Đối chiếu `01/GTGT` ↔ Hóa đơn ↔ Giấy nộp tiền; sinh XML chuẩn HTKK cho `01/GTGT`, `01/KHBS`, `04/GTGT`, `01/CNKD` (Thông tư 50/2026 & Nghị định 68/2026).
5. **Tải hóa đơn trực tiếp Cổng Thuế (`tai-hoa-don.js`):** Tải cả hóa đơn điện tử thường và hóa đơn máy tính tiền (`/sco-query`), đối soát chéo Cổng Thuế với kho nội bộ.
6. **Tính thuế TNDN Luật 67/2025/QH15 (`dashboard-ke-toan.js`):** Bậc thuế suất 15% / 17% / 20%, tự động chuyển lỗ giữa các quý, điều chỉnh `[B4]` / `[C3]`.
7. **Kiểm toán rủi ro Hóa đơn Nhóm 1 & Xuất MISA tự học Nhóm 2 (`kiem-toan-misa.js`):**
   - Ký lệch ngày / vắt kỳ, Tiền mặt $\ge 5$ triệu đ (Luật GTGT 48/2024/QH15) & cộng dồn cùng ngày, Thủng dải số bán ra, Sai tên/ĐC/MST người mua.
   - Định khoản tự học TT200/TT99 & TT133, xuất `.xlsx` chuẩn MISA/FAST.

---

## Phần B. Các Nhóm Nghiệp vụ Kế toán Chuyên sâu Nâng cấp trong Bản V4

### Nhóm 3 — Khớp Sao kê Ngân hàng ↔ Hóa đơn & Quản trị Công nợ (`131` / `331`)
- Đọc sao kê ngân hàng XML ISO 20022 (`camt.052` / `camt.053`) và tệp CSV/TSV sao kê ngân hàng nội địa.
- Tự động dò số hóa đơn, MST/tên đối tác và số tiền chuyển khoản để khớp Ủy nhiệm chi / Báo có với từng hóa đơn mua vào, bán ra.
- Tự động xác nhận điều kiện khấu trừ thuế GTGT cho hóa đơn mua vào $\ge 5$ triệu đồng đã có chứng từ ngân hàng; cảnh báo hóa đơn $\ge 5$ triệu đồng chưa có UNC.
- Lên **Báo cáo Công nợ & Tuổi nợ (Aging Report)** TK `131` (Phải thu khách hàng) và TK `331` (Phải trả nhà cung cấp) theo khung `Trong hạn (0–30 ngày)`, `31–60 ngày`, `61–90 ngày`, `Quá hạn > 90 ngày`.

### Nhóm 4 — Đối chiếu chéo 3 chiều: Lương – Thuế TNCN (`05/KK`, `05/QTT`) – BHXH (`D02-LT`)
- Tổng hợp chỉ tiêu `[16]` (số lao động), `[21]` / `[22]` (tổng TNCT), `[29]` / `[31]` (thuế TNCN đã khấu trừ) qua 4 quý / 12 tháng trên `05/KK-TNCN` và đối chiếu với quyết toán năm `05/QTT-TNCN`.
- Đối chiếu quỹ lương và danh sách lao động trên hồ sơ BHXH XML (`bhxh`) với tờ khai TNCN để phát hiện chênh lệch quỹ lương trước thanh tra.

### Nhóm 5 — Kiểm soát Giá vốn, Cảnh báo "Xuất khống / Âm kho chứng từ" & Lãi gộp mặt hàng
- Phân tích chi tiết dòng hàng `<DSHHDVu>/<HHDVu>` trên toàn bộ hóa đơn mua vào & bán ra của công ty.
- Cảnh báo **Xuất trước ngày Nhập** (mặt hàng xuất bán ngày $T_1$ nhưng hóa đơn đầu vào đầu tiên lập ngày $T_2 > T_1$, hoặc tổng lượng xuất lũy kế vượt tổng lượng nhập tại thời điểm xuất).
- Phân tích **Biên lợi nhuận gộp theo mặt hàng** và cảnh báo mặt hàng bán dưới giá mua bình quân (rủi ro ấn định thuế).

### Nhóm 6 — Hỗ trợ Quyết toán Năm: Kiểm chéo 12 chỉ tiêu BCTC & Sinh nháp `03/TNDN`
- **Kiểm tra chéo 12 chỉ tiêu sống còn trên BCTC XML (TT200 / TT133 / TT99):** Cân đối Tổng Tài sản = Tổng Nguồn vốn, Tiền cuối kỳ CĐKT = Lưu chuyển tiền tệ, Lợi nhuận sau thuế KQKD = biến động LNST chưa phân phối, khớp Thuế GTGT còn được khấu trừ (`133` ↔ `[43]` tờ khai kỳ cuối năm), khớp Doanh thu BCTC ↔ Tổng doanh thu hóa đơn bán ra, v.v.
- **Sinh tệp XML nháp Tờ khai Quyết toán Thuế TNDN (`03/TNDN`):** Tự động tổng hợp Doanh thu, Giá vốn, Chi phí, Lợi nhuận kế toán trước thuế `[A1]`, điều chỉnh tăng `[B4]`, chuyển lỗ `[C3a]`, thuế suất 15%/17%/20% theo Luật 67/2025/QH15 và xuất tệp `.xml` chuẩn cấu trúc HTKK.
