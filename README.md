# YBIM TDTC (Version 0.1.5)
## Giới thiệu
YBIM TDTC (YBIM – Tiến Độ Thi Công) là phần mềm quản lý tiến độ thi công và tài nguyên BIM mạnh mẽ, được xây dựng bằng **WPF**, **.NET 8.0** và giao diện **Fluent Design** theo chuẩn Windows 11.

- **Phiên bản hiện tại:** `v0.1.5`
- **Các tính năng nổi bật:**

  - **Lập & Quản lý Tiến độ (Scheduling):** Xây dựng cấu trúc WBS, thiết lập liên kết logic chuyên sâu (FS, SS, FF, SF, Lag âm/dương), tự động tính toán đường găng (CPM) và khoảng dự phòng (Slack).
  - **Quản lý Danh mục & Phân bổ Tài nguyên:** Hệ thống quản lý chặt chẽ Nhân công, Máy thi công, Vật tư với nguyên lý kiểm soát Năng lực (Max Capacity) chuyên nghiệp giúp cảnh báo vượt tải. Bảng chấm công chi tiết theo ngày làm việc thực tế.
  - **Nhập/Xuất Dữ liệu Đa nền tảng:** Nạp trực tiếp tiến độ từ MS Project (MPP). Đặc biệt: Tính năng **Nhập và Xuất toàn bộ dự án từ Excel mẫu song ngữ** nhanh chóng chỉ với 1-Click. Xuất mã LISP vẽ tiến độ trên AutoCAD siêu tốc độ.
  - **Thuật toán Tối ưu AI (Genetic Algorithm):** Tính năng đột phá tự động san bằng tài nguyên (Máy/Nhân công) và kéo giãn dòng tiền tối ưu thông qua 3 kịch bản đề xuất tự động.
  - **Biểu đồ & Báo cáo Trực quan:** Tự động xuất biểu đồ Tài nguyên đa tuyến (Line Charts với Gradient) với tùy biến hiển thị Đỉnh/Tổng Ca và Giá trị Vật tư chuyên nghiệp. Hệ thống biểu đồ S-Curve Dòng tiền luỹ kế trực quan trên nền OLED Black. Xuất bản vẽ Gantt ra file PDF Vector siêu nét (Super-HD).
  - **Giao diện Tùy biến (Fluent UI):** Hỗ trợ Dark/Light mode, tùy biến màu Accent qua Color Gallery vô hạn, giao diện phẳng Sleek Mode hiện đại kết hợp Dashboard trực quan. Đặc biệt: dải Ribbon được quy hoạch lại gọn gàng, giảm thiểu 60% diện tích chiều ngang tối ưu cho màn hình laptop 15.6 inch, hỗ trợ Segoe Fluent Icons đồng bộ và các tác vụ xóa/hủy liên kết nhanh.
  - **Cấu hình & Lưu tự động (Options & AutoSave):** Cung cấp Menu Tùy chọn chuyên nghiệp theo phong cách MS Project (chuyển đổi Date Format, Office Theme, thông tin tác giả). Tích hợp cơ chế AutoSave ngầm siêu an toàn bảo vệ tiến độ dự án khỏi rủi ro mất điện/đóng đột ngột.
  - **Bảo mật (Security):** Bảo vệ bản quyền với hệ thống HWID kép và mã hoá chống dịch ngược bằng Obfuscar. Cập nhật phiên bản tự động (OTA).
  - **Tối ưu hóa Hiệu năng & Trải nghiệm Cao cấp (v0.1.5 - Mới):**

    - *Lazy Rendering cho SkiaSharp:* Chỉ render các dòng và liên kết nằm trong vùng nhìn thấy (Viewport), tăng tốc Gantt Chart gấp 5 lần, xử lý mượt mà dự án >5000 dòng.
    - *Chế độ Focus Mode (F11):* Ẩn toàn bộ Ribbon và StatusBar để tối đa hóa không gian làm việc, thoát nhanh bằng phím Esc hoặc nút nổi thông minh.
    - *Đa luồng AI & Loading Overlay:* Thuật toán AI chạy ngầm đa luồng kết hợp với Loading Overlay cao cấp làm mờ màn hình và khóa tương tác an toàn.
    - *Crash-Recovery tự động:* Tự động sao lưu tiến độ ra tệp tạm, tự động dọn dẹp file nháp khi thoát an toàn để tránh hộp thoại khôi phục phiền phức.
    - *Interactive Onboarding:* Hướng dẫn tương tác 4 bước bắt mắt sử dụng công nghệ khoét lỗ vector (GeometryGroup EvenOdd) chỉ dẫn trực quan nút bấm.
  - **Tương tác Đa nền tảng Kỹ thuật Cao (v0.1.5 - Nâng cấp):**

    - *Đẩy ngược dữ liệu ra MS Project (MPXJ Export):* Sử dụng động cơ MPXJ chính thống để xuất tệp tin XML MS Project (MSPDI) tương thích 100%, bảo toàn cấu trúc WBS phân cấp thụt lề, liên kết logic (FS/SS/FF/SF + Lag) và gán tài nguyên.
    - *AutoCAD Auto-Layout Plot (c:YBIMPLOT):* Tích hợp lệnh dàn trang tự động chia toàn bộ dải tiến độ thành các Layout A3 chuẩn in ấn chuyên nghiệp, tự vẽ khung tên cá nhân hóa với thông tin tác giả và email đăng ký bản quyền.
    - *Smart Layering (Kiểm soát Layer LISP):* Tạo cấu trúc lớp chuyên biệt (YBIM_Khung, YBIM_ThanhTienDo, YBIM_DuongGang, YBIM_Chu, YBIM_LienKet, YBIM_Leader) với màu sắc và lineweight kỹ thuật định sẵn để dễ hiệu chỉnh.
## Cài đặt
1. Tải tệp zip phát hành từ GitHub Releases: `YBIM_TDTC_v0.1.5.zip`.
2. Giải nén vào thư mục mong muốn, ví dụ `C:\YBIM_TDTC`.
3. Chạy `YBIM_TDTC.exe`. Lần đầu phần mềm sẽ tự động tải các phụ thuộc NuGet cần thiết.
## Cập nhật
Phần mềm sẽ kiểm tra `update.json` trên nhánh `main` để xác định phiên bản mới nhất. Nếu có bản cập nhật, sẽ hiển thị thông báo và cung cấp URL tải xuống.
```json
{
  "Version": "0.1.5",
  "DownloadUrl": "https://github.com/YBIM-sketch/YBIM_TDTC/releases/download/v0.1.5/YBIM_TDTC_v0.1.5.zip",
  "Changelog": "- Tích hợp AI Genetic Algorithm san bằng tài nguyên tự động.\n- Tái cấu trúc và quy hoạch lại dải Ribbon tối ưu không gian hiển thị cho màn hình laptop 15.6 inch (giảm 60% chiều ngang).\n- Tích hợp 5 nâng cấp hiệu năng & UX đỉnh cao: Lazy Rendering Gantt Chart, Focus Mode F11, Đa luồng AI với Loading Overlay, Crash-Recovery tự động, và Interactive Onboarding 4 bước khoét lỗ vector.\n- Khắc phục cấu hình Assembly Search Path của Obfuscar tương thích 100% với môi trường SDK cục bộ."
}

```
## Hướng dẫn sử dụng
- **Tài liệu chi tiết:** `Huong_Dan_Su_Dung_YBIM_TDTC.md` (đã được cập nhật phiên bản `v0.1.5`).
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
## Bản vá v0.1.5 (Sửa lỗi & Tối ưu Ribbon)
* **Sửa lỗi mở tệp `.ybim`:** Nâng cấp cơ chế tuần tự hóa JSON bỏ qua tham chiếu vòng (`IgnoreCycles`), tự động bù đắp và khởi tạo danh sách trống nếu dữ liệu gán tài nguyên, mối quan hệ logic hoặc lịch bị khuyết thiếu (`null-safety`). Thay thế việc nuốt lỗi âm thầm bằng hộp thoại cảnh báo lỗi tường minh.
* **Tối ưu hiển thị Ribbon:** Điều chỉnh chiều rộng tối thiểu (`MinWidth`) của các nhóm "Giao diện" và "Tính toán & Tối ưu" cùng các nút con của chúng, đảm bảo hiển thị nhãn chữ phẳng trên 1 dòng duy nhất, không bị ngắt chữ mất thẩm mỹ, nâng cao tối đa tính premium của ứng dụng trên màn hình laptop 15.6 inch.
* **Đồng bộ hóa Chủ đề & Tối ưu hóa Tương phản (Mới):**
  * *Tích hợp ControlzEx ThemeManager:* Khắc phục triệt để lỗi "chữ trắng trên nền trắng" của thanh tiêu đề Window Chrome và Ribbon Header. Khi chuyển đổi sang chủ đề tối (**Dark Gray / Black**), thanh tiêu đề và dải ruy-băng tự động đồng bộ sang tông màu tối tương ứng (`Dark.Amber` / `Dark.Green`), chữ hiển thị và các nút điều khiển Window hiển thị sắc nét tuyệt đối.
  * *Custom ComboBox ControlTemplate:* Thay thế toàn bộ ComboBox mặc định trong cửa sổ Tùy chọn bằng Template cao cấp tùy chỉnh động. Loại bỏ hoàn toàn màu nền trắng cứng nhắc của Windows, tự động chuyển đổi màu nền ComboBox sang xám tối hoặc đen sâu OLED đồng điệu với màu nền của ứng dụng, chữ hiển thị có độ tương phản tuyệt hảo.
  * *Backstage Tab dynamic background:* Toàn bộ menu tùy chọn Backstage được đồng bộ hóa với hệ tài nguyên động (`Y_SurfaceBackground` và `Y_TextPrimary`), loại bỏ 100% hiện tượng rò rỉ khoảng trắng sáng.
