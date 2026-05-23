# YBIM TDTC (Version 0.1.4)

## Giới thiệu

YBIM TDTC (YBIM – Tiến Độ Thi Công) là phần mềm quản lý tiến độ thi công và tài nguyên BIM mạnh mẽ, được xây dựng bằng **WPF**, **.NET 8.0** và giao diện **Fluent Design** theo chuẩn Windows 11.

- **Phiên bản hiện tại:** `v0.1.4`
- **Các tính năng nổi bật:**
  - **Lập & Quản lý Tiến độ (Scheduling):** Xây dựng cấu trúc WBS, thiết lập liên kết logic chuyên sâu (FS, SS, FF, SF, Lag âm/dương), tự động tính toán đường găng (CPM) và khoảng dự phòng (Slack).
  - **Quản lý Danh mục & Phân bổ Tài nguyên:** Hệ thống quản lý chặt chẽ Nhân công, Máy thi công, Vật tư với nguyên lý kiểm soát Năng lực (Max Capacity) chuyên nghiệp giúp cảnh báo vượt tải. Bảng chấm công chi tiết theo ngày làm việc thực tế.
  - **Nhập/Xuất Dữ liệu Đa nền tảng:** Nạp trực tiếp tiến độ từ MS Project (MPP). Đặc biệt: Tính năng **Nhập và Xuất toàn bộ dự án từ Excel mẫu song ngữ** nhanh chóng chỉ với 1-Click. Xuất mã LISP vẽ tiến độ trên AutoCAD siêu tốc độ.
  - **Thuật toán Tối ưu AI (Genetic Algorithm):** Tính năng đột phá tự động san bằng tài nguyên (Máy/Nhân công) và kéo giãn dòng tiền tối ưu thông qua 3 kịch bản đề xuất tự động.
  - **Biểu đồ & Báo cáo Trực quan:** Tự động xuất biểu đồ Tài nguyên đa tuyến (Line Charts với Gradient) với tùy biến hiển thị Đỉnh/Tổng Ca và Giá trị Vật tư chuyên nghiệp. Hệ thống biểu đồ S-Curve Dòng tiền luỹ kế trực quan trên nền OLED Black. Xuất bản vẽ Gantt ra file PDF Vector siêu nét (Super-HD).
  - **Giao diện Tùy biến (Fluent UI):** Hỗ trợ Dark/Light mode, tùy biến màu Accent qua Color Gallery vô hạn, giao diện phẳng Sleek Mode hiện đại kết hợp Dashboard trực quan.
  - **Cấu hình & Lưu tự động (Options & AutoSave):** Cung cấp Menu Tùy chọn chuyên nghiệp theo phong cách MS Project (chuyển đổi Date Format, Office Theme, thông tin tác giả). Tích hợp cơ chế AutoSave ngầm siêu an toàn bảo vệ tiến độ dự án khỏi rủi ro mất điện/đóng đột ngột.
  - **Bảo mật (Security):** Bảo vệ bản quyền với hệ thống HWID kép và mã hoá chống dịch ngược bằng Obfuscar. Cập nhật phiên bản tự động (OTA).
### YBIM TDTC Pro v0.1.4 - Bản Cập Nhật Tính Năng Đột Phá

Bản cập nhật v0.1.4 mang đến những cải tiến mạnh mẽ về mặt mỹ thuật báo cáo, quy hoạch tối ưu giao diện màn hình và sửa đổi các lỗi logic đa nền tảng:

1. **Quy hoạch dải Ribbon Toolbar siêu gọn gàng (Mới)**:
   - Loại bỏ hoàn toàn các thuộc tính giới hạn chiều rộng tối thiểu (`MinWidth`) cồng kềnh, giúp thu hẹp 50% kích thước bề rộng hiển thị dư thừa của các cụm nút trên Toolbar.
   - Giải phóng không gian làm việc tối đa cho biểu đồ Gantt, phù hợp hoàn hảo trên các màn hình Laptop 15.6 inch.

2. **Cải tiến vượt bậc tính năng xuất báo cáo PDF Vector (Mới)**:
   - *Khắc phục triệt để lỗi chồng đè nhãn ngày tháng*: Tích hợp thuật toán nội suy đo chiều rộng text và tự động ẩn nhãn Ngày bắt đầu (`task.Start`) nếu vị trí quá sát lề bảng dữ liệu thời gian, đảm bảo nét vẽ luôn sạch đẹp 100%.
   - *Tối giản chân trang*: Thay đổi nội dung footer mặc định thành `"Trang ...."` chuyên nghiệp với font Segoe UI xám dịu.
   - *Cố định lề dưới đúng 10mm*: Đảm bảo footer luôn cách mép giấy dưới chính xác 10mm trên cả bản vẽ Gantt lẫn bảng tổng hợp chi phí, phục vụ công tác in ấn hồ sơ kỹ thuật chuyên sâu.

3. **Khắc phục triệt để lỗi xuất mã AutoCAD LISP**:
   - Khắc phục hoàn toàn lỗi cú pháp (dấu phẩy phân tách thập phân trên các máy tính cài đặt vùng Việt Nam `vi-VN`), đảm bảo nạp file `APPLOAD` thành công 100% trên mọi phiên bản AutoCAD.
   - Hướng dẫn vận hành 3 bước trên AutoCAD: `APPLOAD` -> `VETIENDO` -> `YBIMPLOT` để chia tờ và dàn layout in ấn A3 tự động.

4. **Khắc phục lỗi nạp tệp dự án (.ybim)**:
   - Cải tiến cơ chế giải tuần tự hóa JSON bỏ qua tham chiếu vòng (`IgnoreCycles`), xử lý an toàn các giá trị `NaN`/`Infinity` và cơ chế phòng chống lỗi dữ liệu null-safety cho tệp tin cũ.

## Cài đặt

1. Tải tệp zip phát hành từ GitHub Releases: `YBIM_TDTC_v0.1.4.zip`.
2. Giải nén vào thư mục mong muốn, ví dụ `C:\YBIM_TDTC`.
3. Chạy `YBIM_TDTC.exe`. Lần đầu phần mềm sẽ tự động tải các phụ thuộc NuGet cần thiết.

## Cập nhật

Phần mềm sẽ kiểm tra `update.json` trên nhánh `main` để xác định phiên bản mới nhất. Nếu có bản cập nhật, sẽ hiển thị thông báo và cung cấp URL tải xuống.

```json
{
  "Version": "0.1.4",
  "DownloadUrl": "https://github.com/YBIM-sketch/YBIM_TDTC/releases/download/v0.1.4/YBIM_TDTC_v0.1.4.zip",
  "Changelog": "- Tích hợp AI Genetic Algorithm san bằng tài nguyên tự động.\n- Nâng cấp giao diện Ribbon Sleek Mode, hỗ trợ Format Painter.\n- Tối ưu hoá thuật toán tính tiến độ, hỗ trợ biểu đồ S-curve.\n- Nâng cấp Biểu đồ Đường (Line Chart) với Gradient và thanh cấu hình dữ liệu."
}
```

## Hướng dẫn sử dụng

- **Tài liệu chi tiết:** `Huong_Dan_Su_Dung_YBIM_TDTC.md` (đã được cập nhật phiên bản `v0.1.4`).
- **Video hướng dẫn:** Xem playlist **YBIM TDTC** trên YouTube.

## Đóng góp & Liên hệ

- **Báo lỗi / Yêu cầu tính năng:** Mở issue trên GitHub.
- **Liên hệ tác giả:**
  - Zalo: `0917.433.147`
  - Email: `hoangy.dgbo.k52@gmail.com`
  - Youtube: `@Nguyen_Hoang_Y`
  - TikTok: `@hoangycii`

## Giấy phép

Được phát hành dưới **MIT License**.
