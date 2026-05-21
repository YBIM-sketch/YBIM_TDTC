# YBIM TDTC (Version 0.1.3)

## Giới thiệu

YBIM TDTC (YBIM – Tiến Độ Thi Công) là phần mềm quản lý tiến độ thi công và tài nguyên BIM mạnh mẽ, được xây dựng bằng **WPF**, **.NET 8.0** và giao diện **Fluent Design** theo chuẩn Windows 11.

- **Phiên bản hiện tại:** `v0.1.3`
- **Chức năng chính:**
  - Quản lý dự án, công tác, tài nguyên (nhân công, máy thi công, vật tư).
  - Tự động tính toán tiến độ, dòng tiền và biểu đồ Gantt.
  - Xuất báo cáo PDF, Excel, LISP cho AutoCAD.
  - Gửi phản hồi nhanh qua Gmail.
  - Cập nhật tự động qua `update.json`.
  - **Bảo vệ chống dịch ngược** bằng Obfuscar (đã được tích hợp trong quá trình build Release).

## Cài đặt

1. Tải tệp zip phát hành từ GitHub Releases: `YBIM_TDTC_v0.1.3.zip`.
2. Giải nén vào thư mục mong muốn, ví dụ `C:\YBIM_TDTC`.
3. Chạy `YBIM_TDTC.exe`. Lần đầu phần mềm sẽ tự động tải các phụ thuộc NuGet cần thiết.

## Cập nhật

Phần mềm sẽ kiểm tra `update.json` trên nhánh `main` để xác định phiên bản mới nhất. Nếu có bản cập nhật, sẽ hiển thị thông báo và cung cấp URL tải xuống.

```json
{
  "Version": "0.1.3",
  "DownloadUrl": "https://github.com/YBIM-sketch/YBIM_TDTC/releases/download/v0.1.3/YBIM_TDTC_v0.1.3.zip",
  "Changelog": "- Nâng cấp giao diện Ribbon Sleek Mode, hỗ trợ Format Painter, cải thiện xuất PDF Super‑HD.\n- Tối ưu hoá thuật toán tính tiến độ, hỗ trợ biểu đồ S‑curve.\n- Tích hợp Obfuscar để bảo mật mã nguồn."
}
```

## Hướng dẫn sử dụng

- **Tài liệu chi tiết:** `Huong_Dan_Su_Dung_YBIM_TDTC.md` (đã được cập nhật phiên bản `v0.1.3`).
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
