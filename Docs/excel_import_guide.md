# Hướng dẫn nhập file Excel mẫu cho YBIM_TDTC

## 1. Mục đích
File Excel mẫu được dùng để **nhập dữ liệu dự án** (công việc, tài nguyên, thời gian) vào phần mềm YBIM_TDTC một cách nhanh chóng và chuẩn xác.

## 2. Định dạng file
- Định dạng: **`.xlsx`** (Microsoft Excel 2007+).
- Mỗi sheet **chỉ có một loại dữ liệu** và **đặt tên cố định** như dưới đây.

| Sheet name | Mô tả | Cột bắt buộc | Cột tùy chọn |
|------------|------|--------------|--------------|
| **Tasks**  | Danh sách các công việc | `Id`, `WBS`, `Name`, `Start`, `Finish`, `Predecessors` | `Notes`, `IsSummary` |
| **Resources** | Danh sách tài nguyên (Nhân công, Máy, Vật tư) | `Id`, `Name`, `Type`, `Unit` | `MaxCapacity`, `StandardRate` |
| **Assignments** | Phân bổ tài nguyên cho công việc | `TaskId`, `ResourceId`, `AllocatedUnits` | `DailyHours` (nếu có) |

## 3. Định dạng cột chi tiết
### Tasks sheet
| Cột | Kiểu dữ liệu | Mô tả |
|-----|--------------|-------|
| `Id` | `int` | Mã định danh duy nhất của công việc. |
| `WBS` | `string` | Mã WBS (ví dụ: `1.2.3`). |
| `Name` | `string` | Tên công việc. |
| `Start` | `date` (dd/MM/yyyy) | Ngày bắt đầu. |
| `Finish` | `date` (dd/MM/yyyy) | Ngày kết thúc. |
| `Predecessors` | `string` (comma‑separated Id) | Id các công việc tiền nhiệm (ví dụ: `5,7`). |
| `Notes` | `string` (optional) | Ghi chú. |
| `IsSummary` | `bool` (true/false) | Nếu `true` → công việc là tổng hợp (summary). |

### Resources sheet
| Cột | Kiểu dữ liệu | Mô tả |
|-----|--------------|-------|
| `Id` | `int` | Mã định danh tài nguyên. |
| `Name` | `string` | Tên tài nguyên. |
| `Type` | `string` (Labor / Machinery / Material) | Loại tài nguyên. |
| `Unit` | `string` | Đơn vị đo (ngày, ca, kg…). |
| `MaxCapacity` | `double` (optional) | Công suất tối đa (đối với máy). |
| `StandardRate` | `double` (optional) | Đơn giá chuẩn. |

### Assignments sheet
| Cột | Kiểu dữ liệu | Mô tả |
|-----|--------------|-------|
| `TaskId` | `int` | Id của công việc (phải tồn tại trong sheet Tasks). |
| `ResourceId` | `int` | Id của tài nguyên (phải tồn tại trong sheet Resources). |
| `AllocatedUnits` | `double` | Số lượng tài nguyên được phân bổ (đơn vị = `Unit` của tài nguyên). |
| `DailyHours` | `string` (optional, format `dd/MM/yyyy=hh;dd/MM/yyyy=hh…`) | Nếu muốn ghi chi tiết giờ làm việc mỗi ngày, dùng dạng này. |

## 4. Quy tắc nhập dữ liệu
1. **Không để ô trống** trong các cột bắt buộc. Nếu muốn bỏ qua, nhập giá trị mặc định (`0` cho số, `false` cho bool).
2. **Ngày** phải hợp lệ và `Start` ≤ `Finish`.
3. **Predecessors**: chỉ liệt kê Id của các công việc đã tồn tại trong cùng sheet.
4. **Assignments**: mỗi dòng đại diện cho một phân bổ tài nguyên cho một công việc.
5. Khi hoàn tất, lưu file dưới tên **`YBIM_ImportTemplate.xlsx`** và sử dụng nút **“Nhập File TXT/Excel”** trong phần **DANH MỤC TÀI NGUYÊN**.

## 5. Ví dụ mẫu
Bạn có thể tải **file mẫu** (được tạo tự động) từ thư mục **`Docs/Template`** trong dự án. File mẫu có 3 sheet (Tasks, Resources, Assignments) đã được định dạng sẵn.

---
*File này được tạo để hỗ trợ người dùng xây dựng dữ liệu nhập nhanh, giảm lỗi nhập tay và đồng bộ với giao diện Gantt của YBIM_TDTC.*
