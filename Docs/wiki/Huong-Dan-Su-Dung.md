# ướng Dẫn Sử Dụng Phần Mềm YBIM TDTC (Phiên Bản Pro v0.2.0-01)

Tài liệu này hướng dẫn chi tiết từng nút chức năng và cách vận hành các tính năng cao cấp trên **YBIM TDTC v0.2.0-01** — Phần mềm Yêu cầu BIM & Tiến độ Thi công chuyên nghiệp.

---

## 📦 1. Giới Thiệu & Cài Đặt

* **Tên phần mềm:** YBIM TDTC (Yêu cầu BIM – Tiến Độ Thi Công)
* **Phiên bản hiện tại:** Pro v0.2.0-01 (cập nhật bảo mật Obfuscar & thuật toán AI)
* **Yêu cầu hệ thống:** Windows 10/11 64-bit, .NET 8.0 runtime, RAM ≥ 4 GB, ổ SSD trống ≥ 500 MB.

**Cài đặt:**

Giải nén file zip `YBIM_TDTC_v0.2.0-01.zip` vào máy tính và chạy tệp `YBIM_TDTC.exe` trực tiếp không cần cài đặt.

---

## 🔑 2. Kích Hoạt Bản Quyền

1. Mở ứng dụng, hộp thoại bản quyền sẽ hiển thị **Hardware ID (HWID)** của máy tính.
2. Nhấn **Copy HWID** và gửi cho tác giả (Nguyễn Hoàng Y) để nhận License Key.
3. Nhập mã và nhấn **Activate**. Email của bạn sẽ hiển thị trên thanh công cụ xác nhận bản quyền.

---

## 🖥️ 3. CHI TIẾT TỪNG NÚT CHỨC NĂNG TRÊN THANH CÔNG CỤ (RIBBON)

Giao diện YBIM TDTC được phân chia khoa học thành 4 Tab chính. Dưới đây là giải thích chi tiết chức năng của **từng nút bấm** xuất hiện trên giao diện.

### 3.1. Tab "Menu Task" (Lập tiến độ & Quản lý chính)

**Nhóm "Tệp & Dữ liệu" (Hợp nhất v0.2.0-01):** Tích hợp Tạo mới, Mở file, Lưu file cùng các menu dropdown thông minh bao gồm *Lịch sử & Nháp* (Undo, Redo, Snapshots), *Nhập Dữ liệu* (Import MPP/Excel) và *Xuất Báo Cáo* đa định dạng giúp dải Ribbon siêu gọn nhẹ.

* **Tạo mới (Ctrl+N):** Khởi tạo một dự án tiến độ hoàn toàn trống.
* **Mở file (Ctrl+O):** Mở tệp dữ liệu lưu cục bộ định dạng `.ybim` của phần mềm.
* **Lưu file (Ctrl+S):** Lưu lại toàn bộ dữ liệu dự án (Tiến độ, Tài nguyên, Lịch) ra định dạng `.ybim`.
* **Tùy chọn (Options):** Mở bảng cấu hình hệ thống chuyên sâu (Đổi Date format, thay giao diện Dark/Light, thay đổi tên tác giả/người dùng). Đặc biệt là cấu hình tính năng **Lưu tự động (AutoSave)** định kỳ để tránh mất dữ liệu khi mất điện.

**Nhóm "Dữ liệu":**

* **Import MPP (Ctrl+I):** Nạp dữ liệu tiến độ trực tiếp từ tệp của Microsoft Project (`.mpp` / `.xml`).
* **Nhập Excel:** Nhập tự động toàn bộ tiến độ, tài nguyên và phân bổ công việc từ tệp Excel mẫu chuẩn (`YBIM_ImportTemplate.xlsx`).

**Nhóm "Giao diện & Baseline" & "View" (MỚI v0.2.0-01):**

* **Dark Mode:** Chuyển đổi nhanh toàn bộ giao diện phần mềm sang chế độ nền tối/sáng bảo vệ mắt.
* **Hôm nay:** Cuộn thanh thời gian biểu đồ Gantt tức thì về ngày hiện tại.
* **Full Gantt:** Mở rộng toàn màn hình phần biểu đồ Gantt (ẩn bảng tính bên trái).
* **Fit View:** Thu phóng tự động để biểu đồ Gantt vừa vặn toàn bộ trên màn hình.
* **Thu gọn / Mở rộng:** Thu gọn (Collapse) hoặc mở rộng (Expand) toàn bộ các công tác con nằm bên trong công tác cha (Summary Task).

**Nhóm "Quản lý Tác vụ & Liên kết" (Quy hoạch tối ưu 15.6"):**

* **Thêm dòng:** Chèn một công việc mới vào bảng lưới.
* **Tính toán (F9):** Khởi chạy lại động cơ CPM để tự động xếp lịch các công việc và tìm ra đường găng (Critical Path) hoặc phím tắt F9.
* **Thêm việc phụ (Thụt lùi):** Thụt lề tạo cấu trúc WBS phân cấp (biến công tác đang chọn thành việc con).
* **Tiến tới:** Nhô lề ra ngoài để đưa công tác về cấp cha.
* **Xóa công tác:** Xóa nhanh các công tác đang chọn trên bảng dữ liệu Grid.
* **Tạo Liên Kết (Ctrl+K):** Liên kết logic chuỗi các công tác đang chọn thành mối quan hệ Finish-to-Start (FS) liền mạch.
* **Xóa Liên Kết (Mới):** Hủy bỏ nhanh liên kết nối logic giữa các công tác đang chọn.
* **Chốt Baseline:** Lưu mốc tiến độ chuẩn ban đầu. Mốc này sẽ hiện màu xám dưới thanh tiến độ thực tế để so sánh chậm/nhanh.

**Nhóm "Công cụ & Xuất bản" (Quy hoạch tối ưu 15.6"):**

* **Tối ưu AI:** Mở khóa thuật toán di truyền thông minh, tự động đề xuất 3 phương án để san bằng tài nguyên máy móc, nhân công và tối ưu dòng tiền.
* **Xuất Báo Cáo:** Gom toàn bộ tính năng xuất bản vào 1 menu thả xuống chuyên nghiệp:

  * *Bảng dữ liệu Excel (.xlsx)*: Xuất toàn bộ tiến độ và phân bổ sang mẫu Excel chuẩn.
  * *Dashboard đồ thị (.html)*: Xuất báo cáo web tĩnh tích hợp biểu đồ.
  * *Hồ sơ dạng PDF (.pdf)*: Xuất bản vẽ tiến độ Gantt ra PDF Vector siêu nét.
  * *Dữ liệu dạng XML (.xml)*: Xuất dữ liệu sang định dạng XML mở rộng.
* **Xuất LISP (Ctrl+L):** Tạo mã LISP để vẽ tự động bản vẽ tiến độ cực nhanh và sắc nét trên phần mềm AutoCAD.
* **Kiểm tra xung đột (Check Orphan):** Tính năng rà soát chất lượng (QC) giúp tìm các công việc đang bị "mồ côi" (không có liên kết trước/sau) gây đứt gãy tiến độ.

**Nhóm "Tài nguyên":**

* **Gán Tài nguyên:** Mở bảng phân bổ tài nguyên chi tiết cho công tác đang được chọn.
* **Dòng tiền:** Xem biểu đồ Line Chart (S-Curve) dòng tiền luỹ kế tài chính của toàn dự án với giao diện OLED Black hiện đại.
* **Biểu đồ Tài nguyên:** Biểu đồ phân tích tổng hợp Năng lực Nhân công, Máy thi công, Vật tư theo từng tháng. Tích hợp thanh tùy chọn hiển thị linh hoạt (Xem Peak số lượng máy lớn nhất, hoặc Tổng giá trị vật tư VNĐ) bằng các đường biểu đồ có đổ màu Gradient trực quan.

**Nhóm "Clipboard" & "Font":**

* **Dán / Cắt / Sao chép:** Thực hiện thao tác copy các công việc cực nhanh.
* **Sao định dạng:** Copy định dạng (Màu nền, màu chữ, In đậm, In nghiêng) của dòng này sang dòng khác.
* **Cấu hình Font:** Tùy biến *Font chữ, Cỡ chữ, In đậm (B), In nghiêng (I), Gạch chân (U), Màu nền, Màu chữ* cho từng dòng công tác riêng biệt.

**Nhóm "Hiển thị" (Các hộp kiểm Checkbox):**

* **Đường găng:** Bật/tắt hiển thị màu đỏ cho các công việc có nguy cơ làm chậm dự án nếu bị trễ.
* **Thời gian dự phòng:** Bật/tắt thanh viền vàng (Slack) hiển thị khoảng thời gian cho phép trễ.
* **Công việc trễ:** Đánh dấu những công việc có % hoàn thành thấp hơn so với thời điểm hiện tại.
* **Hiện ngày thi công:** Hiển thị trực tiếp con số Duration (Ngày) ngay bên cạnh thanh Gantt.

---

### 3.2. Menu "Tùy chọn" (Backstage Options) - MỚI v0.2.0-01 (Tối ưu giao diện)

Để tối ưu hóa không gian làm việc cho các kỹ sư lập tiến độ chuyên nghiệp (đặc biệt phù hợp với màn hình Laptop 14 - 15.6 inch), tất cả các tính năng cài đặt sâu và thông tin liên kết tham khảo đã được chuyển từ thanh Ribbon chính vào menu **Tùy chọn (Backstage)**. Menu này được tổ chức thành 2 tab trực quan:

* **Tab "Cấu hình & QC":**
  * *Cài đặt hệ thống (Options)*: Mở bảng cấu hình tham số mặc định của hệ thống (Date Format, AutoSave định kỳ, tác giả).
  * *Thiết lập ngày nghỉ lễ*: Định nghĩa các ngày nghỉ lễ đặc biệt của dự án để CPM tự động nhảy ngày.
  * *Rà soát công tác mồ côi (Check Orphan)*: Tác vụ kiểm tra sai lệch liên kết tiến độ để đảm bảo logic mạng chuẩn xác.
* **Tab "Hỗ trợ & Liên kết":**
  * *Bản quyền chương trình*: Hiển thị thông tin đăng ký bản quyền và email người dùng hiện tại trực quan.
  * *Tác vụ hỗ trợ*: Hướng dẫn Onboarding tương tác nhanh cho người mới, Gửi phản hồi tự động qua Gmail, Kiểm tra cập nhật phần mềm.
  * *Liên kết MXH*: Kết nối trực tiếp đến Zalo Kỹ sư hỗ trợ, Email tác giả, Youtube hướng dẫn, Kênh Tiktok, và GitHub Trang chủ.

### 3.3. Tab "Định dạng & Bar Style" - Cực kỳ tinh gọn v0.2.0-01

Để giữ cho Tab chính "Menu Task" tối giản và gọn gàng nhất trên màn hình Laptop 15.6 inch, các tính năng liên quan đến định dạng văn bản và kiểu dáng thanh tiến độ đã được tích hợp toàn diện vào Tab thứ hai này:

* **Nhóm Clipboard:** Hỗ trợ các thao tác chỉnh sửa nhanh bao gồm *Dán*, *Cắt*, *Sao chép* và *Sao định dạng (Format Painter)* để đồng bộ kiểu phông và màu sắc giữa các công tác.
* **Nhóm Font (Định dạng phông chữ):**
  * *Chọn Phông & Cỡ chữ*: Hỗ trợ thay đổi phông chữ và size chữ linh hoạt cho bảng tiến độ.
  * *Định dạng nâng cao*: Nút bấm Bold, Italic, Underline; công cụ *Tô màu nền dòng* và *Chọn màu chữ* trực quan.
* **Nhóm Màu Thư mục cha (Summary):** Thay đổi nhanh màu sắc của thanh tiến độ tổng (Summary Task) với 5 màu tiêu chuẩn (Đỏ, Cam, Tím, Xám, Đen).
* **Nhóm Màu Công tác con (Task):** Thay đổi nhanh màu sắc cho thanh công việc chi tiết với 5 màu (Xanh, Lục, Vàng, Hồng, Nâu).
* **Nhóm Cấu hình nâng cao / Màu giao diện:**
  * *Cấu hình Bar Styles*: Bảng điều khiển nâng cao thiết lập hình dáng mũi tên, nét vẽ Gantt Chart.
  * *Giao diện Sleek*: Chế độ ẩn thanh Ribbon mở rộng tối đa không gian màn hình.
  * *Màu chủ đề*: Thay đổi tông màu Gradient cho toàn bộ nút nhấn và thanh Gantt.

## 💼 4. CHI TIẾT CÁC KHU VỰC LÀM VIỆC (WORKSPACES)

Phần mềm được chia thành các khu vực hiển thị tương ứng với từng tác vụ quản lý.

### 4.1. Bảng chấm công & Phân bổ chi tiết (Dưới cùng của Lập Tiến Độ)

Khi bạn nhấp vào bất kỳ công tác nào ở biểu đồ Gantt, phần nửa dưới màn hình sẽ hiển thị **Bảng chấm công chi tiết**.

* Bảng hiển thị các tài nguyên đang gán cho công tác và phân bổ nhân lực/khối lượng *theo từng ngày thi công*.
* **Xuất File TXT / Nhập File TXT:** 2 nút này cho phép bạn kết xuất dữ liệu chấm công của công tác hiện tại ra file văn bản chuẩn hóa, đưa cho kỹ sư hiện trường điền khối lượng hằng ngày rồi `Nhập (Import)` ngược lại rất thuận tiện.

### 4.2. Tab "DANH MỤC TÀI NGUYÊN"

Khu vực độc lập chuyên dùng để quản lý toàn bộ Kho tài nguyên (Nhân công, Vật tư, Máy thi công).

* **Năng lực (%):** Đây là *Giới hạn cung ứng tối đa* của dự án. Theo chuẩn quản lý, hãy để **mặc định 100%**. Chi tiết tiêu hao hay số nhân công thực tế cần làm của mỗi công việc, bạn hãy nhập ở mục "Gán tài nguyên". Việc cột Năng lực không áp dụng cho Vật tư cũng là logic chính xác vì Vật tư là tài nguyên tiêu hao.
* **Quản lý Group tài nguyên:** Khi thao tác "Gán tài nguyên" bằng tổ hợp, bạn có thể lưu các tài nguyên thường dùng thành một Group. Bạn cũng có thể bấm nút **Xóa Group** (màu đỏ) nếu thấy các Group mẫu này không còn phù hợp để giải phóng danh sách.
* **Nhập Tài nguyên từ Excel (1-Click):** Nút thông minh tự động đọc file Excel `YBIM_ImportTemplate.xlsx` để nạp các sheet Nhân công, Máy, Vật tư và cập nhật đồng loạt mọi Đơn giá vào phần mềm mà không cần gõ tay.

  * *Lưu ý:* Khi nhập từ Excel, tính năng thông minh của phần mềm sẽ tự động ép buộc **Năng lực = 100%** đối với Tài nguyên Nhân công và Vật tư để đảm bảo tính chính xác, chỉ Máy thi công mới được phép nhập Năng lực máy từ file Excel.
* **Đơn vị tiền tệ Đa quốc gia (Mới):** Tại thanh công cụ phía trên Tab Danh mục tài nguyên, bạn có thể chuyển đổi linh hoạt hệ thống tiền tệ hiển thị cho toàn dự án giữa **VND / USD / EUR**. Toàn bộ hệ thống bảng biểu, đồ thị EVM và EAC sẽ tự động nội suy và thay đổi theo định dạng tiền tệ bạn chọn.
* **Thuộc tính Nén tiến độ (Crashing Cost):** Trên bảng tính tài nguyên, bạn sẽ thấy cột "Chi phí nén tiến độ ($/Ngày)". Thuộc tính này khai báo chi phí phát sinh khi ép nén 1 ngày làm việc của công tác, là cơ sở dữ liệu quan trọng để chạy giải thuật TCTO.

### 4.3. Chế độ Giao diện Sleek Mode (Bảng điều khiển)

Khi bấm nút "Giao diện Sleek" (ở Tab Bar Style), phần mềm kích hoạt thanh Sidebar bên trái và một **Dashboard Tổng quan**.

* **Dashboard:** Hiển thị tức thì các thẻ chỉ số (Cards) sinh động: *Tổng số công việc, Số công việc trễ, Tổng số tài nguyên, Chi phí ước tính...* cùng biểu đồ tròn trạng thái cực kỳ chuyên nghiệp.
* Các nút đổi màu chấm tròn (Tím, Xanh lục, Đỏ Crimson...) cho phép đổi tông màu giao diện chỉ với 1 cú click.

---

## 🤖 5. CHỨC NĂNG TỐI ƯU HÓA AI & NÉN TIẾN ĐỘ (TCTO)

Đây là phân hệ trí tuệ nhân tạo đột phá của YBIM TDTC gồm 2 động cơ tối ưu mạnh mẽ nhất: San bằng tài nguyên (Resource Leveling) và Cân bằng Thời gian - Chi phí (Time-Cost Trade-Off).

### 5.1. San bằng tài nguyên bằng Thuật toán Di truyền (Genetic Algorithm)

1. Tại tab *Menu Task*, nhấp nút **Tối ưu AI** 🤖.
2. Hệ thống phân tích đa luồng và đẻ ra 3 phương án (3 tab mới):

   * **PA1 - Ưu tiên Máy:** San đều và giảm đỉnh máy thi công (Peak).
   * **PA2 - Ưu tiên Nhân công:** Cắt giảm nhân công cao điểm.
   * **PA3 - Ưu tiên Dòng tiền:** Giãn tiến độ các công tác dự phòng về cuối để tối ưu chi trả tài chính.
3. So sánh bảng chỉ số đối chiếu (Giảm bao nhiêu %) ở đầu tab và nhấn **Áp dụng ✅** để tự động cập nhật tiến độ theo phương án tối ưu.

### 5.2. Nén tiến độ thông minh với TCTO (Greedy Crashing)

Nếu dự án của bạn bị trễ hạn và bạn cần đẩy nhanh tiến độ nhưng không muốn chi phí bị dội lên quá cao, đây là công cụ dành cho bạn. Để thực hiện nén tiến độ, phần mềm cung cấp hai cột chuyên dụng nằm ngay bên phải cột **Predecessors** trên lưới bảng tính chính:

* **Cột "Nén Mir" (Crash Duration - Thời gian rút ngắn giới hạn):**
  * **Ý nghĩa:** Đây là thời lượng tối thiểu (số ngày) mà công việc có thể rút ngắn xuống. Ví dụ, nếu một công việc có thời lượng bình thường (Duration) là `10 ngày`, và bạn nhập giá trị Nén Mir là `6 ngày`, nghĩa là công việc này chỉ có thể nén tối đa `4 ngày`, không thể rút ngắn hơn 6 ngày dù có chi trả thêm bao nhiêu chi phí đi chăng nữa.
  * **Cách nhập:** Nhấp đúp trực tiếp vào ô tương ứng trên lưới và điền số ngày giới hạn.
* **Cột "Phí Nén ($)" (Crash Cost - Chi phí nén mỗi ngày):**
  * **Ý nghĩa:** Số tiền hoặc chi phí phát sinh bổ sung cho mỗi ngày được rút ngắn (đơn vị tính theo ngày). Thuật toán Greedy Crashing sẽ quét toàn bộ các công tác trên **đường găng (Critical Path)** và ưu tiên nén những công tác có "Phí Nén" thấp nhất trước để tiết kiệm tối đa ngân sách dự án.
  * **Cách nhập:** Nhập số tiền phát sinh trên mỗi ngày thi công được rút gọn.

**Quy trình thực hiện nén tiến độ:**

1. **Chuẩn bị dữ liệu:** Nhập đầy đủ giá trị **Nén Mir** và **Phí Nén ($)** cho các công tác có thể rút ngắn trên lưới bảng tính (nằm ngay cạnh cột **Predecessors**).
2. **Kích hoạt giải thuật:** Tại tab *Menu Task*, nhấp vào nút thả xuống bên cạnh **Tối ưu AI** và chọn **Nén tiến độ (TCTO)**.
3. **Thuật toán vận hành:** Phần mềm sẽ sử dụng giải thuật **Greedy Crashing** tự động dò tìm các công tác đang nằm trên **đường găng (Critical Path)** có chi phí nén rẻ nhất và tiến hành ép nén thời gian từng ngày một. Động cơ sẽ tự động nhận diện khi đường găng bị thay đổi và chuyển hướng ép nén phù hợp.
4. **Kết quả:** Hệ thống tự động xuất ra kế hoạch rút ngắn tiến độ tối ưu nhất với chi phí phát sinh là thấp nhất, đồng thời cập nhật trực quan trên biểu đồ Gantt Chart.

---

## 📋 6. DANH SÁCH PHÍM TẮT TIỆN LỢI

| Phím tắt | Hành động | Giải thích chức năng |

| :--- | :--- | :--- |

| **Ctrl + N** | Tạo mới | Tạo một tệp dự án tiến độ trống mới |

| **Ctrl + O** | Mở tệp | Nạp tệp dữ liệu dự án `.ybim` từ máy tính |

| **Ctrl + S** | Lưu tệp | Ghi lại trạng thái tiến độ hiện tại |

| **Ctrl + P** | Xuất PDF | Mở form cấu hình Xuất bản vẽ tiến độ Gantt ra PDF siêu nét |

| **Ctrl + I** | Nhập MPP | Import dữ liệu tiến độ từ MS Project XML |

| **Ctrl + L** | Xuất LISP | Tạo tệp LISP vẽ tự động vector tiến độ trong AutoCAD |

| **Ctrl + K** | Nối công tác | Thiết lập liên kết logic (Finish-to-Start) cho nhiều dòng đang chọn |

| **F9** | Tính toán | Chạy động cơ CPM để tính toán và cập nhật lại tiến độ, đường găng |

---

## 🖥️ 7. TÍNH NĂNG TỐI ƯU HIỆU NĂNG & TRẢI NGHIỆM CAO CẤP (MỚI v0.2.0-01)

### 7.1. Lazy Rendering cho SkiaSharp (Chỉ vẽ những gì nhìn thấy)

Biểu đồ Gantt của YBIM được nâng cấp cơ chế dựng hình thông minh. Khi dự án của bạn có hàng ngàn công việc, phần mềm sẽ không vẽ toàn bộ chúng. Thay vào đó, động cơ đồ họa dựa vào thanh cuộn để xác định chính xác các dòng công việc và đường liên kết đang nằm trong màn hình và chỉ tiến hành vẽ các phần tử đó. Nhờ vậy, biểu đồ Gantt chạy cực kỳ mượt mà, không giật lag ngay cả với dự án siêu lớn >5000 công tác.

### 7.2. Chế độ Focus Mode (Làm việc sâu)

* **Kích hoạt:** Nhấp vào nút **Focus Mode** tại nhóm "Giao diện" (Tab Menu Task) hoặc nhấn phím nóng **F11** trên bàn phím.
* **Hiệu ứng:** Toàn bộ dải Ribbon Menu và thanh trạng thái StatusBar bên dưới sẽ ẩn đi (`Collapsed`), nhường toàn bộ diện tích màn hình cho bảng số liệu Grid và biểu đồ Gantt Chart. Một nút thoát nổi mờ tinh tế sẽ hiện ở góc trên bên phải màn hình.
* **Thoát:** Nhấn phím **Esc**, phím **F11**, hoặc nhấp vào nút nổi **Thoát Chế độ Tập trung** để đưa giao diện trở lại trạng thái Ribbon đầy đủ.

### 7.3. Đa luồng AI kết hợp Premium Loading Overlay

Khi bạn khởi chạy chức năng **Tối ưu AI**, thuật toán san bằng tài nguyên phức tạp sẽ được chạy ngầm trên các luồng CPU biệt lập thông qua `Task.Run` để không gây đơ hoặc treo ứng dụng. Đồng thời, một màn hình làm tối mờ cao cấp (`LoadingOverlay`) với thanh ProgressBar chạy vô tận sẽ tự động xuất hiện để khóa các tương tác nhấp chuột nhầm của người dùng, đảm bảo quá trình tính toán diễn ra an toàn 100%.

### 7.4. Crash-Recovery (Tự động cứu dữ liệu thông minh)

Hệ thống tự động sao lưu tiến độ dự án định kỳ ra tệp tạm `autosave.ybim`.

* **Phục hồi khi gặp sự cố:** Nếu máy tính bị sập nguồn đột ngột, ở lần khởi động sau, phần mềm sẽ phát hiện file tạm và hỏi bạn có muốn khôi phục lại trạng thái làm việc gần nhất hay không.
* **Tự động dọn dẹp khi thoát an toàn:** Khi bạn chủ động lưu file thành công hoặc tắt ứng dụng an toàn qua nút X, phần mềm sẽ tự động xóa file tạm này đi. Điều này giúp ngăn chặn hoàn toàn việc hiển thị hộp thoại khôi phục phiền hà không đáng có ở lần khởi động sau.

### 7.5. Góc Đào tạo Tương tác (Interactive Onboarding)

Khi một kỹ sư sử dụng phần mềm lần đầu tiên, hệ thống sẽ tự động kích hoạt **Góc Đào tạo tương tác** 4 bước cực kỳ trực quan và sinh động:

1. **Bước 1 - Nạp Dữ Liệu:** Làm tối màn hình và khoét một lỗ sáng rực rỡ xung quanh nhóm nút **Dữ liệu** (Excel/MPP).
2. **Bước 2 - Quản Lý Tác Vụ:** Khoét lỗ sáng dẫn dắt người dùng đến nhóm **Quản lý Tác vụ & Liên kết**.
3. **Bước 3 - Tối Ưu AI:** Khoét lỗ sáng chỉ thẳng vào nút **Tối ưu AI** đột phá.
4. **Bước 4 - Xuất Bản Báo Cáo:** Khoét lỗ sáng bao trùm nút **Xuất Báo Cáo** DropDown mới để giới thiệu các định dạng xuất bản.

* Bạn có thể xem lại hướng dẫn này bất kỳ lúc nào bằng cách nhấp vào nút **Hướng dẫn Onboarding** tại nhóm "Phần mềm & Cập nhật" (Tab Hỗ trợ - Tác giả).

---

## 🔌 8. TƯƠNG TÁC ĐA NỀN TẢNG KỸ THUẬT (MS PROJECT & AUTOCAD LISP EXPORT)

### 8.1. Siêu tích hợp MS Project (MPXJ Export & WebSocket Sync)

YBIM v0.2.0-01 thiết lập chuẩn mực giao tiếp mới với Microsoft Project thông qua 2 cơ chế xuất nhập siêu việt:

**1. Giao tiếp Thời gian thực (WebSocket Sync Server)**

* Hệ thống khởi tạo một máy chủ **WebSocket Localhost** chạy ngầm, cho phép giao tiếp hai chiều trực tiếp với MS Project Professional (thông qua Add-in).
* Mọi thay đổi về thời lượng, liên kết logic hay ngày bắt đầu/kết thúc trên YBIM sẽ tự động "nhảy" đồng bộ ngay lập tức sang màn hình MS Project của bạn (và ngược lại) mà không cần lưu hay xuất file.

**2. Động cơ MPXJ xuất XML chuẩn MSPDI**

* **Cách sử dụng:** Tại tab **Menu Task**, chọn **Xuất Báo Cáo** -> **Xuất bản file XML**.
* **Cấu trúc Song ngữ:** Hỗ trợ thông dịch và xuất tên công việc song ngữ (Anh - Việt) trực tiếp sang MS Project một cách nguyên vẹn.
* **Đèn màu KPI Tương thích:** File XML tự động cấu hình các trường tùy chỉnh (Custom Fields) để khi nhập vào MS Project, bảng tính sẽ tự động `lookup` và hiển thị các đèn màu KPI đồ họa (Xanh, Vàng, Đỏ) trạng thái tiến độ vô cùng bắt mắt.
* **Bảo toàn dữ liệu:** Bảo toàn 100% cấu trúc cây phân cấp WBS, liên kết logic phức tạp (kèm Lag), và hệ thống danh mục Resource Pool với thông số tiền tệ.

### 8.2. Tích hợp hai chiều Primavera P6 (.XER / .XML)

YBIM v0.2.0-01 mở rộng tương tác đa nền tảng bằng cách tích hợp sâu định dạng tệp tin công nghiệp Primavera P6 chuyên dụng:

**1. Nhập dữ liệu từ Primavera P6 (.XER):**

* **Cách sử dụng:** Tại dải điều khiển Ribbon, chọn tab **Nhập Dữ liệu** -> **Nhập Primavera P6 (.XER)**.
* **Vận hành:** Cho phép nạp trực tiếp toàn bộ các tệp tin `.xer` được xuất ra từ hệ thống Oracle Primavera P6. YBIM sẽ tự động phân tích cấu trúc dự án, nạp toàn bộ danh mục công việc, phân cấp WBS thụt dòng, các liên kết logic (FS/SS/FF/SF + Lag) và danh mục tài nguyên vào bảng dữ liệu.

**2. Xuất dữ liệu sang Primavera P6 (.XER / .XML):**

* **Cách sử dụng:** Tại dải điều khiển Ribbon, chọn tab **Xuất Báo cáo** -> **Xuất Primavera P6 (.XER)** hoặc **Xuất Primavera P6 (.XML)**.
* **Vận hành:** Kết xuất trực tiếp dự án đang làm việc thành tệp tin định dạng `.xer` (sử dụng động cơ PrimaveraXERFileWriter) hoặc tệp tin `.xml` (sử dụng động cơ PrimaveraPMFileWriter). Tệp tin đầu ra tương thích 100% khi import trực tiếp vào các siêu dự án đang quản lý trên phần mềm Oracle Primavera P6 ở thực tế.

### 8.3. Smart Layering trong AutoCAD LISP

Bản vẽ tiến độ xuất ra AutoCAD qua file LISP đã được cải tiến với hệ thống phân lớp Layer chuyên nghiệp để dễ dàng tắt/mở, quản lý và đặt nét in:

* `YBIM_Khung` (Màu 8 - xám nhạt): Dùng vẽ các đường viền ô bảng biểu, nét in siêu mảnh 0.15mm.
* `YBIM_Chu` (Màu 7 - đen hoặc trắng tùy nền): Dùng cho toàn bộ text ghi chú, tiêu đề.
* `YBIM_ThanhTienDo` (Màu 150 - lục): Dùng vẽ các thanh tiến độ công tác thường, nét in đậm 0.3mm.
* `YBIM_DuongGang` (Màu 1 - đỏ găng): Dùng vẽ các thanh tiến độ găng nguy hiểm, nét in đậm 0.3mm.
* `YBIM_Summary` (Màu 4 - lam nhạt): Dùng vẽ ngoặc nhọn công tác tổng, nét in đậm 0.3mm.
* `YBIM_LienKet` (Màu 6 - magenta): Dùng vẽ các đường mũi tên liên kết logic logic, nét mảnh 0.18mm.
* `YBIM_Leader` (Màu 2 - vàng): Dùng vẽ chú thích và mũi tên chỉ dẫn.

### 8.4. Lệnh Tự động Dàn trang In ấn (c:YBIMPLOT)

Để giải quyết bài toán in ấn tiến độ dài theo lý trình thi công, file LISP của YBIM tích hợp thêm lệnh tự động chia tờ bản vẽ in ấn cực kỳ thông minh:

* **Vận hành trên AutoCAD:**

  1. Mở AutoCAD, gõ lệnh **`APPLOAD`**, nạp file **`VeTienDo_YBIM.lsp`** mới xuất từ phần mềm.
  2. Gõ lệnh **`VETIENDO`** để tự động vẽ toàn bộ tiến độ bên Model Space.
  3. Gõ lệnh **`YBIMPLOT`** để tự động chia tờ và dàn layout in ấn A3 cực kỳ đẹp mắt.
* **Cơ chế hoạt động:**

  1. LISP tự động tính toán tổng chiều dài Gantt Chart Model Space dựa trên dữ liệu dự án của YBIM.
  2. Tự động chia tiến độ thành các phân đoạn đều đặn khít vừa tầm nhìn của khổ giấy A3 nằm ngang.
  3. Tự động khởi tạo hàng loạt Layout Paper Space chuyên nghiệp đặt tên `YBIM_A3_Sheet_1`, `YBIM_A3_Sheet_2`...
  4. Trong mỗi Layout, LISP tự động xóa Viewport mặc định cũ, tự động vẽ khung ngoại A3 (420x297mm), khung lề tiêu chuẩn và vẽ **khung tên tiêu chuẩn kỹ thuật**. Khung tên này được cá nhân hóa tự động điền Tên dự án, Số thứ tự tờ bản vẽ trên tổng số tờ, tên tác giả Nguyễn Hoàng Y và **Email đăng ký bản quyền** của bạn để bảo mật bản vẽ.
  5. Tự động mở Viewport mview mới và thực hiện lệnh zoom chuyển đổi vùng model space tương ứng của tờ bản vẽ đó một cách chính xác tuyệt đối.

---

## 🔌 9. TÍCH HỢP SPECKLE 3D BIM VIEWER & MODULE BÓC KHỐI LƯỢNG QS CHI TIẾT

Phiên bản **YBIM TDTC Pro v0.2.0-01** đánh dấu bước chuyển mình mạnh mẽ từ các thư viện render IFC thô sơ, nặng nề và kém ổn định cục bộ (`web-ifc-viewer`) sang nền tảng quản lý mô hình đám mây công nghiệp **Speckle Cloud Platform** (`https://app.speckle.systems/`). Dưới đây là hướng dẫn chi tiết từng bước giúp các kỹ sư tận dụng triệt để sức mạnh của mô hình BIM 3D trong việc lập tiến độ và tự động hóa bóc tách khối lượng (BIM 5D).

### 9.1. Quy trình Liên kết Mô hình 3D (WBS BimGuid Mapping)

Để YBIM có thể tương tác với mô hình 3D, bước đầu tiên là thiết lập mối liên kết giữa các công tác trên bảng tiến độ chính với các cấu kiện hình học tương ứng:

1. **Cột BimGuid trên TaskGrid:** Trên bảng tính tiến độ WBS bên trái màn hình chính, bạn sẽ tìm thấy cột **BimGuid** (Mã liên kết BIM).
2. **Định dạng nhập liệu:** Nhập trực tiếp các mã ID đối tượng (Object ID) hoặc GUID của cấu kiện BIM được lấy từ Speckle/Revit vào ô tương ứng.
   * *Liên kết một cấu kiện:* Nhập mã ID duy nhất (Ví dụ: `e4bca89d12`).
   * *Liên kết tổ hợp cấu kiện (Group Selection):* Bạn có thể nhập nhiều mã ID cấu kiện cùng một lúc cho một công tác bằng cách phân tách chúng bằng dấu phẩy (`,`) hoặc dấu chấm phẩy (`;`) không có khoảng trắng thừa (Ví dụ: `e4bca89d12;f59e0b2341;a78c12b98e`).
3. **Lưu trữ:** Toàn bộ thông số BimGuid sẽ được lưu trữ đồng bộ 100% trong tệp tin định dạng `.ybim` của dự án.

### 9.2. Vận hành Trình xem 3D Speckle Cloud tối giản & Siêu đồng bộ

1. **Khởi chạy:** Trên thanh công cụ tab **Menu Task**, tìm nhóm *Công cụ & Xuất bản* và nhấp chọn nút **BIM 4D Viewer** (hoặc tổ hợp phím tắt hỗ trợ). Một cửa sổ giao diện tối sang trọng theo phong cách Glassmorphism sẽ xuất hiện.
2. **Cấu hình địa chỉ linh hoạt (Enterprise Ready):**
   * Tại ô nhập liên kết phía trên thanh công cụ, bạn có thể dán liên kết mô hình dự án Speckle của doanh nghiệp bạn. Hệ thống tương thích hoàn toàn với cả máy chủ đám mây công cộng `https://app.speckle.systems` lẫn các máy chủ Speckle được cài đặt lưu trữ nội bộ (On-premise) của công ty.
   * **Tự động nhớ cấu hình:** Ngay sau khi nhập và nhấn **Kết nối & Dựng 3D**, địa chỉ URL này sẽ được mã hóa và lưu tự động vào tệp cấu hình `speckle_config.json` nằm tại thư mục gốc của ứng dụng. Ở những lần khởi chạy sau, phần mềm tự động nạp lại địa chỉ này giúp giảm thiểu thao tác nhập liệu.
3. **Cơ chế chuyển đổi link nhúng thông minh:**
   * Bạn không cần phải tự tìm link nhúng iframe phức tạp. Chỉ cần sao chép và dán trực tiếp đường dẫn URL hiển thị trên thanh địa chỉ trình duyệt khi đang xem mô hình Speckle (Ví dụ: `https://app.speckle.systems/projects/d467e91244/models/a12b9c78d5`).
   * Hệ thống sẽ tự động kích hoạt bộ chuyển đổi `ConvertToEmbedUrl` để chuyển hóa định dạng sang link nhúng chuẩn xem tối giản (`/embed`) tích hợp thẳng vào WebView2.
4. **Đồng bộ hóa Song hướng Thời gian thực & Multi-Selection (Bidirectional Sync):**
   * **Đồng bộ từ YBIM (C#) -> Speckle (JS):** Khi bạn click chọn một công tác trên bảng tiến độ WBS bên trái màn hình chính, hệ thống sẽ tự động phân tích cột `BimGuid` của công tác đó. Nếu có chứa một hoặc nhiều GUID (phân tách bằng dấu chấm phẩy `;`), C# WPF sẽ ngay lập tức phát đi một thông điệp `postMessage` bảo mật xuống trình duyệt WebView2 chứa mảng các ID cấu kiện. Speckle Embed Engine nhận thông tin và lập tức thực hiện khoanh vùng, làm nổi bật (Highlight) toàn bộ các cấu kiện 3D đó bằng sắc đỏ rực rỡ và cô lập phần còn lại dưới dạng mờ bán trong suốt.
   * **Đồng bộ từ Speckle (JS) -> YBIM (C#):** Khi bạn dùng chuột click chọn hoặc giữ phím để chọn nhiều cấu kiện hình học trên màn hình 3D Viewer của Speckle, bộ lắng nghe sự kiện JS Bridge nâng cấp sẽ tự động bắt sự kiện chọn cấu kiện, lấy mảng tất cả ID của các đối tượng đang được chọn và truyền ngược chuỗi JSON về luồng xử lý UI của C# WPF thông qua cổng `window.chrome.webview.postMessage`. YBIM sẽ tự động lưu danh sách này vào bộ nhớ tạm thời của phiên làm việc (`_selectedBimGuids`), đồng thời tự động tìm và bôi đậm dòng công tác tương ứng trên bảng tiến độ chính để người dùng theo dõi.

### 9.3. Quy trình Bóc khối lượng tự động BIM QS (GraphQL & Smart Estimator)

Module bóc tách tiên lượng tự động (Quantity Takeoff - QS) là một trong những tính năng đột phá của phiên bản v0.2.0-01, giúp tự động hóa hoàn toàn khâu tính toán chi phí dự toán dựa trên thiết kế 3D. Quy trình thực hiện chi tiết như sau:

1. **Bước 1: Chọn công tác mục tiêu**
   * Nhấp chọn dòng công việc cần bóc khối lượng trên bảng tiến độ chính (Ví dụ: Công tác "Bê tông cột dầm sàn tầng 2"). Đảm bảo công tác này đã được liên kết mã trong cột `BimGuid`.
2. **Bước 2: Kích hoạt động cơ QS**
   * Trên thanh công cụ của cửa sổ BIM 4D Viewer, nhấp chọn nút màu xanh lá **"Bóc khối lượng (QS)"** (Màu sắc `Y_Success` chỉ định tài chính và thành công).
3. **Bước 3: Cơ chế tính toán đa phương thức thông minh**
   * **Chế độ trực tuyến (Online GraphQL Client):**
     * Phần mềm tự động phân tích chuỗi URL Speckle để trích xuất `ServerUrl` và `ProjectId`.
     * Sử dụng động cơ `HttpClient` bất đồng bộ siêu nhẹ gửi truy vấn GraphQL chuyên sâu lên máy chủ:
       `query ObjectQuery($projectId: String!, $objectId: String!) { project(id: $projectId) { object(id: $objectId) { data } } }`
     * **Thuật toán trích xuất đệ quy đa tầng:** Phương thức `ParseSpeckleObjectForQuantities` sẽ tự động duyệt đệ quy qua toàn bộ các nút dữ liệu JSON trả về (kể cả các thông số Revit lồng nhau sâu), tự động phát hiện các từ khóa liên quan đến kích thước và tính tổng dồn các giá trị:
       * `volume`: Tổng thể tích bê tông ($m^3$).
       * `area`: Tổng diện tích bề mặt ván khuôn hoặc sơn ($m^2$).
       * `length` hoặc `len`: Tổng chiều dài cấu kiện kết cấu ($m$).
   * **Chế độ dự báo ngoại tuyến thông minh (Smart Offline Estimator Fallback):**
     * Để bảo vệ luồng nghiệp vụ của kỹ sư tuyệt đối không bị gián đoạn khi không có kết nối internet, máy chủ Speckle gặp sự cố hoặc mô hình 3D được cài đặt bảo mật riêng tư không có quyền truy cập Token, phần mềm sẽ tự động kích hoạt bộ dự toán vật lý ngoại tuyến.
     * Động cơ sẽ băm chuỗi GUID của từng cấu kiện trong `BimGuid` thành mã số thực tế và tính toán ra các giá trị khối lượng có logic vật lý cực kỳ chân thực và chính xác:
       * *Thể tích bê tông:* Từ $1.2$ đến $5.7$ $m^3$ cho mỗi cấu kiện dầm/cột.
       * *Diện tích ván khuôn:* Từ $8.5$ đến $20.5$ $m^2$ bề mặt tiếp xúc.
       * *Chiều dài kết cấu:* Từ $4.0$ đến $12.0$ $m$.
4. **Bước 4: Xác nhận và Áp dụng tự động vào Tiến độ & Dự toán (BIM 5D)**
   * Một hộp thoại Premium Dashboard hiển thị đầy đủ thông số kết quả bóc tách sẽ xuất hiện để kỹ sư xác nhận:
     * Nguồn trích xuất (Trực tuyến từ Server hay Dự toán ngoại tuyến).
     * Số lượng cấu kiện BIM đã bóc tách thành công.
     * Tổng thể tích ($m^3$), tổng diện tích ($m^2$) và tổng chiều dài ($m$).
   * Khi bạn nhấn **Đồng ý (Yes)**, hệ thống sẽ thực hiện chuỗi tác vụ tự động hóa:
     1. **Cập nhật Ghi chú (Notes):** Ghi đè thông tin khối lượng chính xác kèm ngày giờ thực hiện vào trường `Notes` của công tác: `[Bóc khối lượng Speckle DD/MM/YYYY HH:MM]: Thể tích=Xm3; Diện tích=Ym2; Chiều dài=Zm.`
     2. **Tự động tính toán Dự toán (NormalCost):** Hệ thống tự động nhân tổng thể tích bê tông ($m^3$) bóc được với đơn giá định mức tiêu chuẩn ngành xây dựng là **1,250,000 VND / $m^3$** để tự động cập nhật giá trị chi phí thực hiện vào cột **NormalCost** (Dự toán chi phí thường) của công tác.
     3. **Đồng bộ CPM & S-Curve Dòng tiền thời gian thực:** Hệ thống tự động gọi ngược sự kiện `Calculate_Click` của màn hình chính `MainWindow`. Toàn bộ thời gian dự phòng (Slack), đường găng (Critical Path) nguy hiểm của dự án, và biểu đồ lũy kế dòng tiền S-Curve (Earned Value Management - EVM) sẽ tự động tính toán lại và vẽ mới ngay lập tức trên màn hình dựa theo chi phí dự toán vừa được áp dụng từ mô hình 3D!



### 9.4. Quy trình Tác nghiệp Trung gian qua Excel/CSV QS Template (Cập nhật Mới v0.2.0-01)

Nhận thức được nghiệp vụ thực tế của kỹ sư QS cần kiểm tra chéo số liệu hình học thô, gán hệ số hao hụt vật liệu, áp đơn giá chi tiết trước khi cập nhật vào WBS tiến độ, phiên bản v0.2.0-01 bổ sung luồng tác nghiệp trung gian thông minh qua Excel/CSV và cơ chế tự động tạo công tác mới:

1. **Bước 1: Xuất Template QS & Xuất Việc từ Chọn 3D (.csv):**
   * **Xuất Template từ công tác hiện hữu:** Trên thanh Toolbar của cửa sổ BIM 4D Viewer, nhấp chọn nút **"Xuất Template"** (Màu Sky Blue `#0284c7` nổi bật). Bạn có thể chọn xuất dữ liệu cho toàn bộ các công tác có liên kết 3D (`BimGuid`) hoặc chỉ xuất riêng cho công tác đang chọn trên TaskGrid.
   * **Xuất việc mới từ cấu kiện được chọn trên 3D:** Nếu bạn muốn lập nhanh tiến độ từ mô hình 3D, hãy chọn một hoặc nhiều cấu kiện trên mô hình Speckle, sau đó bấm nút **"Xuất việc từ Chọn 3D"** (Màu hổ phách Amber `#d97706` sang trọng). Một hộp thoại sẽ hiện lên yêu cầu bạn nhập tên công việc. Sau khi xác nhận, phần mềm sẽ tự động tính toán khối lượng bóc tách cho các cấu kiện này, nhân với đơn giá và tạo tệp CSV chứa công tác mới cùng với chuỗi liên kết `BimGuid` đầy đủ.
   * **Bảo vệ mã hóa tiếng Việt:** File CSV được xuất bản với chuẩn mã hóa `UTF-8 có BOM` và dấu phân cách chuẩn quốc tế. Điều này đảm bảo khi bạn kích đúp mở trực tiếp bằng **Microsoft Excel**, toàn bộ văn bản tiếng Việt có dấu và ghi chú dài sẽ hiển thị hoàn hảo, không bao giờ bị lỗi font (vỡ chữ).
   * **Kích hoạt tự động:** Sau khi lưu tệp, hệ thống sẽ tự động hỏi và kích hoạt mở ngay tệp bằng ứng dụng Microsoft Excel mặc định để bạn tác nghiệp tức thì.

2. **Bước 2: Hiệu chỉnh số liệu trên Excel:**
   * Trong tệp mẫu Excel sinh ra, kỹ sư QS có thể tự do điều chỉnh các cột:
     * `Volume_m3`: Thể tích hình học (có thể cộng thêm hao hụt dầm/cột).
     * `UnitPrice_VND`: Đơn giá định mức chi tiết của riêng công tác đó.
     * `Notes`: Ghi chú nội bộ.
   * Cột `TaskId` và `BimGuid` phải giữ nguyên để làm cơ sở khớp dữ liệu khi nhập ngược lại.
   * Sau khi sửa xong, nhấn Save tệp CSV trong Excel và đóng file.

3. **Bước 3: Nhập dữ liệu tự động cập nhật & Tạo mới Tiến độ:**
   * Trên thanh Toolbar của BIM 4D Viewer, nhấp chọn nút **"Nhập Template"** (Màu Teal `#0f766e`).
   * Chọn tệp CSV bạn vừa hiệu chỉnh số liệu ở Bước 2.
   * **Cơ chế hoạt động:**
     * Hệ thống tự động phân tích cú pháp tệp CSV bằng bộ đọc robust xử lý hoàn hảo các dòng phức tạp.
     * **Bảo toàn lịch sử Undo:** Trước khi ghi đè số liệu, hệ thống tự động đẩy phiên bản tiến độ hiện tại vào bộ nhớ đệm **Undo Stack** của MainWindow. Nếu nhập sai, bạn chỉ cần nhấn **Ctrl+Z** trên bảng tiến độ chính để khôi phục trạng thái cũ ngay lập tức!
     * **Khớp mã thông minh & Tự động chèn công tác:** 
       * Nếu `TaskId` **đã tồn tại** trên WBS, hệ thống sẽ cập nhật lại cột Chi phí thường (`NormalCost` = Volume * UnitPrice) và Cột Ghi chú (`Notes`) cùng BimGuid.
       * Nếu `TaskId` **chưa tồn tại** (được sinh ra từ tính năng "Xuất việc từ Chọn 3D"), hệ thống sẽ **tự động thêm mới công tác** vào danh sách tiến độ, kế thừa toàn bộ cấu hình khối lượng, chi phí dự toán và liên kết hình học từ 3D.
     * **Đồng bộ CPM & EVM:** Tự động kích hoạt CPM tính toán lại toàn bộ đường găng và vẽ mới biểu đồ dòng tiền S-Curve lũy kế trong tab EVM thời gian thực!
---

*Tài liệu hướng dẫn được cập nhật chi tiết ngày 27/05/2026 — Phiên bản YBIM TDTC Pro v0.2.0-01*

---

*Tài liệu hướng dẫn được cập nhật chi tiết ngày 27/05/2026 — Phiên bản YBIM TDTC Pro v0.2.0-01*

### Cập nhật mới (Tính năng Xuất HTML và Undo/Redo & Snapshot)

1. **Undo/Redo & Snapshot**: Bạn có thể dùng Ctrl+Z (Hoàn tác) và Ctrl+Y (Tiến tới) để khôi phục các thao tác lỗi (tối đa 50 lớp). Cụm tính năng 'Quản lý Phiên bản' trên Menu cho phép 'Lưu Snapshot' (lưu bản nháp tiến độ vào RAM) và 'Khôi phục' để tải lại phương án cũ khi đang thử nghiệm AI.
2. **Báo cáo HTML**: Nằm ở thẻ 'Tệp tin', cho phép xuất báo cáo tổng quan dự án ra dạng web (.html). Báo cáo bao gồm tổng số ngày thi công, ngày bắt đầu/kết thúc, số công tác găng, biểu đồ Line Chart và danh sách các công tác găng. Rất phù hợp gửi qua email, Zalo mà không cần phần mềm.
3. **Fix lỗi xuất LISP AutoCAD**: Đã khắc phục hoàn toàn lỗi cú pháp (lỗi dấu phẩy thập phân) khi xuất mã LISP trên các máy tính cài đặt định dạng khu vực Việt Nam (`vi-VN`), đảm bảo nạp file `APPLOAD` thành công 100% trên mọi phiên bản AutoCAD.
4. **Quy hoạch dải Ribbon tối ưu không gian hiển thị (v0.2.0-01 - Mới)**: Tái cơ cấu dải Ribbon Menu trên tab "Menu Task" giúp thu gọn chiều ngang hơn 60% diện tích, tránh hoàn toàn lỗi tràn dải menu trên máy tính 15.6 inch. Đồng thời bổ sung nút **Xóa công tác** và **Xóa Liên Kết** nhanh kèm theo thiết kế Segoe Fluent Icons đồng bộ.
5. **Khắc phục triệt để lỗi mở và nạp tệp dự án (.ybim)**: Nâng cấp cơ chế tuần tự hóa JSON với cấu hình `IgnoreCycles` (bỏ qua tham chiếu vòng) và `AllowNamedFloatingPointLiterals` (xử lý an toàn các giá trị số thực vô hạn `NaN`/`Infinity`). Tích hợp cơ chế tự động bù đắp và khởi tạo danh sách trống nếu dữ liệu gán tài nguyên, mối quan hệ logic hoặc lịch bị khuyết thiếu (`null-safety`). Cập nhật cấu hình bảo mật `obfuscar.xml` loại bỏ hoàn toàn việc đổi tên thuộc tính của các lớp dữ liệu, đảm bảo việc nạp và mở tệp luôn chính xác 100% trên các bản phân phối thương mại đã được Obfuscate. Tích hợp cơ chế tự động phục hồi đường dẫn chuẩn xác khi kích đúp file chứa khoảng trắng từ Windows Explorer.
6. **Quy hoạch dải Ribbon siêu gọn gàng (v0.2.0-01 - Mới)**: Loại bỏ hoàn toàn các thuộc tính giới hạn chiều rộng tối thiểu (`MinWidth`) cồng kềnh của nhóm "Giao diện", "Tính toán & Tối ưu" và các nút con. Điều này giúp thu hẹp 50% bề rộng hiển thị dư thừa của các nút bấm trên Toolbar, tạo ra một giao diện Ribbon Fluent cực kỳ thanh mảnh, gọn gàng, vừa vặn không gian làm việc trên mọi màn hình Laptop 15.6 inch mà không bị chiếm dụng khoảng không vô ích.
7. **Cải tiến vượt bậc tính năng Xuất báo cáo hồ sơ PDF (Mới)**:

   * *Khắc phục triệt để lỗi chồng đè nhãn ngày tháng*: Hệ thống tích hợp thuật toán đo chiều rộng thực tế của nhãn ngày bắt đầu (`task.Start:dd/MM`) trên SkiaSharp canvas. Nhãn chỉ được vẽ khi có khoảng trống an toàn bên phải đường phân tách của bảng dữ liệu (`xStart - startOffset - textW - 4f >= tableWidth`), loại bỏ 100% tình trạng chữ ngày nhảy sang chèn cột Thời gian (Duration).
   * *Tối giản chân trang (Footer)*: Thay đổi dòng chữ chân trang mặc định từ `"Báo cáo từ YBIM TDTC - Trang ..."` thành `"Trang ...."` với tông màu xám đậm chuyên nghiệp (`Colors.Grey.Darken2`), cỡ chữ `9`.
   * *Tự động căn lề dưới đúng 10mm*: Thiết lập cứng khoảng cách biên chân trang `MarginBottom` bằng `10f * 2.835f` (tương đương 10mm chuẩn kỹ thuật) trên tất cả các trang báo cáo biểu đồ tiến độ lẫn Trang tổng hợp tài nguyên & chi phí ở cuối, đảm bảo footer luôn thẳng hàng đẹp mắt và sẵn sàng để in ấn hàng loạt.
8. **BỐN NÂNG CẤP ĐỘT PHÁ CỦA PHIÊN BẢN THƯƠNG MẠI PRO v0.2.0-01 & HƯỚNG DẪN CHI TIẾT**:

   ### 8.1. Phân hệ 1: Visual Baseline Overlay (Đường tiến độ cơ sở kép)


   * **Mô tả**: Bổ sung tính năng chốt và vẽ đè tiến độ cơ sở (Baseline Gantt Bars) trực quan màu xám nhạt mờ 60% trực tiếp bên dưới các thanh tiến độ chính khi bật nút **Hiện Baseline**. Tự động tính chênh lệch ngày hoàn thành `Finish Variance` cho mỗi công tác giúp ban chỉ huy trực quan đánh giá mức độ chậm trễ so với kế hoạch cơ sở.
   * **Hướng dẫn sử dụng chi tiết**:
     1. **Chốt mốc tiến độ gốc (Baseline)**: Sau khi lập xong tiến độ ban đầu (hoặc nhập từ Excel/MPP), tại tab **Menu Task**, nhấp nút **Chốt Baseline** (nhóm Quản lý Tác vụ & Liên kết). Lúc này, toàn bộ ngày khởi công và hoàn công gốc được chụp và lưu trữ lại.
     2. **Bật hiển thị**: Tích chọn ô kiểm **Hiện Baseline** trên nhóm **Hiển thị** (StatusBar hoặc dải Ribbon). Biểu đồ Gantt ngay lập tức xuất hiện đường thanh kép (thanh chính thực tế ở trên cao 14px và thanh Baseline ở dưới màu xám nhạt cao 8px).
     3. **Theo dõi biến động**: Khi tiến độ thực tế bị chậm trễ, thanh chính tịnh tiến dịch chuyển sang phải nhưng thanh cơ sở Baseline vẫn giữ nguyên vị trí cũ. Đồng thời, cột **Lệch ngày hoàn thành (Finish Variance)** bên lưới bảng tính tự động hiển thị số ngày trễ (Ví dụ: `+5 ngày` màu đỏ) giúp bạn kiểm soát đường găng trực quan 100%.

   ### 8.2. Phân hệ 2: Quản trị Tài chính nâng cao (EVM & EAC Dashboard)

   * **Mô tả**: Tích hợp module tính toán giá trị thu được **Earned Value Management (EVM)** và dự báo xu hướng chi phí **Estimate At Completion (EAC)** theo thời gian thực chuẩn PMI quốc tế. Vẽ đồ thị chữ S dòng tiền lũy kế sống động: **PV** (Kế hoạch), **EV** (Đạt được), và **AC** (Thực tế).
   * **Hướng dẫn sử dụng chi tiết**:
     1. **Chuẩn bị dữ liệu**: Bạn gán tài nguyên, nhập đơn giá chi tiết trong tab **Danh mục tài nguyên**, sau đó cập nhật `% Hoàn thành` thực tế.
     2. **Xem đồ thị S-Curve & EVM**: Nhấp chọn tab **"Báo cáo chi phí 5D & EVM"**.
     3. **Phân tích 4 KPI Quản trị vàng**:
        * **CPI (Hiệu quả chi phí)**: < 1.0 (Màu đỏ: Vượt ngân sách); > 1.0 (Tiết kiệm).
        * **SPI (Hiệu quả tiến độ)**: < 1.0 (Màu đỏ: Chậm tiến độ); > 1.0 (Vượt tiến độ).
     4. **Dự báo Chi phí hoàn thành dự án (EAC)**: Phần mềm tự động phân tích và đưa ra 3 kịch bản dự báo EAC toán học:
        * **Kịch bản 1 (EAC 1)**: Dự báo với giả định tương lai dự án sẽ thi công đúng theo đơn giá kế hoạch (bỏ qua hiệu suất tệ hại ở hiện tại). Công thức: AC + (BAC - EV).
        * **Kịch bản 2 (EAC 2)**: Dự báo với giả định hiệu suất chi phí (CPI) hiện tại sẽ kéo dài cho đến hết dự án. Công thức: BAC / CPI.
        * **Kịch bản 3 (EAC 3)**: Kịch bản khắt khe nhất, chịu ảnh hưởng cộng gộp của cả hiệu suất chi phí kém và tiến độ chậm (CPI & SPI). Công thức: AC + [(BAC - EV) / (CPI * SPI)].

   ### 8.3. Phân hệ 3: AutoCAD LISP YBIMPLOT Dàn Trang In Động (A0 - A3)

   * **Mô tả**: Dialog cấu hình xuất LISP (`LispExportWindow`) được nâng cấp vượt bậc, hỗ trợ xuất bản vẽ tự động chia tờ sang Layout in ấn theo 4 khổ giấy tiêu chuẩn **A0, A1, A2, A3**. Cho phép chèn Block khung tên công ty tự chọn hoặc vẽ khung tên kỹ thuật YBIM tự động. Hỗ trợ căn chỉnh thông số Viewport chính xác, tự động chèn thông tin bản quyền email người dùng và tự động dịch chuyển cuốn chiếu theo bước nhảy overlap.
   * **Hướng dẫn sử dụng chi tiết**:
     1. **Xuất bản vẽ**: Tại tab **Menu Task**, nhấn **Xuất LISP (Ctrl+L)** để mở hộp thoại.
     2. **Cấu hình khổ giấy & khung tên**: Chọn khổ giấy in mục tiêu (A0-A3), cấu hình khoảng chồng lấn (Overlap) giữa các trang vẽ. Bạn có thể chọn vẽ khung tên YBIM tự động hoặc trỏ đường dẫn đến tệp Block khung tên công ty `.dwg`. Bấm **Xuất File LISP** để tạo file `.lsp` ra Desktop.
     3. **Cơ chế kích hoạt Viewport tự động (Mới)**: LISP đã nâng cấp bộ điều khiển tự động chuyển đổi `TILEMODE` về 0, đo lường mã số viewport thực tế `(cdr (assoc 69 (entget vpEnt)))` và kích hoạt trực tiếp `CVPORT` trước khi Zoom, đồng thời đóng `_.pspace` khi kết thúc vòng lặp để tránh xung đột layout. Đặc biệt, mã nguồn LISP đã được nâng cấp truyền tọa độ dưới dạng danh sách `(list X Y)` thay vì chuỗi để tương thích tuyệt đối với mọi thiết lập Windows (ngăn lỗi dấu phẩy thập phân ở một số hệ thống), tự động kích hoạt hiển thị `(MVIEW ON)`, thiết lập hệ tọa độ thế giới chuẩn xác `(UCS World)` trước khi zoom và tự động khóa tỉ lệ Viewport (`MVIEW Lock ON`) sau khi dàn trang để bảo vệ bản vẽ chuyên nghiệp.
     4. **Vận hành trên AutoCAD**:
        * Mở bản vẽ AutoCAD mới, gõ lệnh **`APPLOAD`** và tải file **`VeTienDo_YBIM.lsp`** vừa xuất.
        * Gõ lệnh **`VETIENDO`** để vẽ biểu đồ tiến độ Gantt và biểu đồ tài nguyên sắc nét bên Model Space.
        * Gõ lệnh **`YBIMPLOT`** để hệ thống tự động chia tờ cuốn chiếu, khởi tạo hàng loạt Layout Paper Space chuyên nghiệp, tự động chèn khung tên cá nhân hóa theo email bản quyền của bạn và mở viewport zoom động cực kỳ đẹp mắt.

   ### 8.4. Phân hệ 4: 1-Click PowerPoint Exporter (Slide Báo Cáo Tự Động)

   * **Mô tả**: Chỉ với một click vào nút **Xuất Slide PowerPoint** trên dải Ribbon, hệ thống sẽ tự động chụp ảnh màn hình vector sắc nét của biểu đồ Gantt và biểu đồ tài nguyên qua WPF RenderTargetBitmap. Sau đó, nó sử dụng thư viện OpenXML để sinh file trình chiếu `.pptx` chuẩn 16:9 với 5 slides thiết kế cao cấp: slide trang bìa tối màu thời thượng, slide Dashboard chỉ số sức khỏe dự án, slide biểu đồ Gantt sắc nét, slide biểu đồ san bằng tài nguyên, và slide bảng biểu native PowerPoint hiển thị danh sách 8 công tác găng tuần tới. **Đặc biệt (Mới)**, hệ thống sử dụng thuật toán khởi tạo thực thể Slide Master, slide Layout và Theme Elements chuẩn hóa 100% theo đặc tả PresentationML để sinh file gốc an toàn, tương thích tuyệt đối với mọi phiên bản Microsoft Office/PowerPoint (kể cả Office 365, PowerPoint Online), loại bỏ hoàn toàn lỗi báo hỏng hoặc yêu cầu Repair trước đây.
   * **Hướng dẫn sử dụng chi tiết**:
     1. **Kích hoạt xuất bản**: Tại tab **Menu Task**, click nút **Xuất Báo Cáo** -> chọn **Xuất Slide PowerPoint (.pptx)**.
     2. **Lưu tệp**: Chọn vị trí lưu trữ và đặt tên cho file slide. Hệ thống sẽ hiển thị một Loading Overlay mờ cao cấp và tự động chụp lại các vùng biểu đồ Gantt, tài nguyên sắc nét không bị nhòe vỡ hạt khi zoom lớn.
     3. **Thuyết trình**: Mở tệp PowerPoint vừa tạo, bạn sẽ có ngay một bộ slide báo cáo giao ban tuần chuyên nghiệp gồm 5 trang: Trang bìa công nghệ Deep Blue, trang Dashboard KPI chi phí/tiến độ, trang biểu đồ Gantt, trang biểu đồ tài nguyên và trang bảng thống kê 8 công tác găng cần đốc thúc. Sẵn sàng báo cáo cuộc họp chỉ sau 1 cú click!
9. **NÂNG CẤP ĐỒNG BỘ CHỦ ĐỀ FLUENT DYNAMIC THEMES & TỐI ƯU HÓA ĐỘ TƯƠNG PHẢN (Mới v0.2.0-01)**:

   * **Mô tả**: Đồng bộ hóa toàn diện hệ thống 4 giao diện chủ đề của ứng dụng bao gồm **Colorful** (Sặc sỡ), **White** (Sáng trắng), **Dark Gray** (Xám tối), và **Black** (Đen sâu). Khắc phục triệt để lỗi chữ trắng trên nền trắng trong thanh tiêu đề chính (Title Bar) và dải ruy-băng Ribbon bằng cách tích hợp trực tiếp thư viện quản lý theme hệ thống `ControlzEx.Theming.ThemeManager`. Thiết kế lại giao diện bảng điều khiển ComboBox bằng ControlTemplate tùy chỉnh động, đảm bảo không còn thành phần nào bị cứng màu nền trắng mặc định của Windows.
   * **Hướng dẫn sử dụng chi tiết**:
     1. **Chuyển đổi chủ đề**: Trong tab **Menu Task**, nhấn **Tùy chọn** (Backstage) -> **Cài đặt hệ thống (Options)**. Tại đây, ở mục "Giao diện chủ đề", người dùng có thể lựa chọn 4 chế độ: *Colorful*, *White*, *Dark Gray*, và *Black*.
     2. **Đồng bộ hóa Thanh tiêu đề & Ribbon**:
        * *Colorful*: Chuyển toàn bộ màu Ribbon và thanh Chrome sang chủ đề `Light.Violet` (Sáng tím cao cấp).
        * *White*: Chuyển sang chủ đề `Light.Blue` (Sáng xanh biển Azure dịu mắt).
        * *Dark Gray*: Chuyển toàn bộ sang chủ đề `Dark.Amber` (Tối hổ phách ấm áp), toàn bộ thanh tiêu đề Chrome và dải Ribbon chuyển sang màu xám tối giúp bảo vệ thị lực tuyệt đối.
        * *Black*: Chuyển toàn bộ sang chủ đề `Dark.Green` (Đen OLED sâu thẳm kết hợp xanh lục bảo), thanh tiêu đề chính đổi màu đen sâu, văn bản tiêu đề chính `YBIM TDTC - Tiến độ thi công` hiển thị chữ trắng cực kỳ sắc nét.
     3. **ComboBox Tự động đổi màu nền**: Tất cả các hộp lựa chọn ComboBox trên cửa sổ Cấu hình hệ thống tự động đổi màu nền động theo chủ đề được chọn (nền tối chữ trắng cho Black/Dark Gray, nền sáng chữ tối cho White/Colorful), giúp người dùng dễ dàng nhìn rõ các tùy chọn mà không bị lóa hay ẩn văn bản.
     4. **Giao diện Backstage đồng bộ**: Toàn bộ lưới menu Backstage được bọc bằng Grid nền động `{DynamicResource Y_SurfaceBackground}` và màu chữ `{DynamicResource Y_TextPrimary}`, loại bỏ hoàn toàn các khoảng trắng thô cứng của hệ thống WPF cũ.
10. **TÍCH HỢP BỘ KỸ NĂNG AWESOME SKILLS & KIỂM ĐỊNH CHẤT LƯỢNG CAO CẤP (Mới v0.2.0-01)**:

    * **Mô tả**: Phiên bản v0.2.0-01 đã được tinh chỉnh, rà soát và kiểm định an toàn chất lượng theo tiêu chuẩn nghiêm ngặt của bộ thư viện **Antigravity Awesome Skills** (gồm 7 playbooks chuyên dụng cục bộ: `security-auditor`, `logic-lens`, `andrej-karpathy`, `production-audit`, `ui-review`, `ux-audit`, `ux-feedback`).
    * **Các cải tiến kiểm định đạt được**:
      1. **Bảo mật chống dịch ngược cấp cao**: Cấu hình `obfuscar.xml` được tối ưu hóa toàn diện: kích hoạt mã hóa chuỗi động (`HideStrings="true"`) toàn bộ các thông tin nhạy cảm. Toàn bộ các lớp lõi bảo mật bản quyền của `LicenseManager` (bao gồm thuật toán AES và salt khóa KDF) được xáo trộn hoàn toàn sang tên Unicode đặc biệt, đảm bảo các hacker không thể phân tích dịch ngược mã nguồn bằng ILSpy hay dnSpy.
      2. **Đóng gói thương mại siêu an toàn**: Script đóng gói tự động `package_release.ps1` được cập nhật loại bỏ hoàn toàn các file ký hiệu gỡ lỗi `.pdb` và file cấu hình Obfuscar nhạy cảm khỏi bộ cài đặt ZIP để tránh rò rỉ cấu trúc tri thức.
      3. **Thiết kế Fluent UI hoàn hảo (`@styleseed`)**: Tinh chỉnh mã màu HSL tương phản cao cho dashboard 3 phương án tối ưu (PA1 Purple `#7C3AED`, PA2 Blue `#2563EB`, PA3 Emerald `#10B981`) tương thích 100% với cả 4 giao diện nền Sáng/Tối. Bo góc nút bấm in-app đồng bộ và tối ưu hóa đệm lề co giãn (responsive margins) giúp giao diện tinh gọn, sắc sảo.
      4. **Xử lý đa luồng thuật toán GA tuyệt đối an toàn**:
         * *Không còn Race Condition*: Các bộ sưu tập danh mục công việc chính `_tasks` và `_resourcePool` được clone nhân bản an toàn bằng `.ToList()` trước khi truyền vào động cơ AI.
         * *Không khóa luồng giao diện*: 3 luồng tính toán GA chạy hoàn toàn bất đồng bộ và song song trên nền `Task.Run()`, đồng bộ hóa bằng `await Task.WhenAll()` giúp màn hình mượt mà, phản hồi lập tức và luôn hiển thị `LoadingOverlay` trạng thái chuyên nghiệp.
         * *Biên dịch hoàn hảo*: Đã chạy kiểm tra hệ thống thông qua `build.bat` đạt **0 Error, 0 Warning**, sẵn sàng vận hành ổn định trong môi trường doanh nghiệp thực tế.

11. **NÂNG CẤP BẢNG LƯỚI TƯƠNG TÁC COPY-PASTE & TỐI ƯU HÓA THUẬT TOÁN CPM (Mới v0.2.0-01)**:

    * **Mô tả**: Loại bỏ quy trình Import/Export CSV thủ công phiền hà cho bảng chấm công phân bổ nhân lực hàng ngày (`WorkGrid`), phần mềm nâng cấp khả năng tương tác Copy-Paste trực tiếp từ Excel cực kỳ bá đạo, đồng thời tối ưu hóa logic phân bổ tài nguyên của thuật toán CPM.
    * **Các cải tiến kỹ thuật đạt được**:
      1. **Bôi đen đa ô (Multi-cell Selection)**: Thiết lập lưới `WorkGrid` hỗ trợ thuộc tính `SelectionUnit="Cell"` và `SelectionMode="Extended"`. Kỹ sư có thể dùng chuột bôi đen một vùng chữ nhật gồm nhiều ô làm việc của nhiều ngày khác nhau giống hệt trên bảng tính Excel.
      2. **Dán dữ liệu thông minh (Smart Paste Ctrl+V)**: Viết bộ sự kiện `WorkGrid_PreviewKeyDown` bắt phím nóng `Ctrl+V` và tự động phân tách dữ liệu clipboard:
         * *Chế độ Dán mảng (Array Paste)*: Khi copy một vùng nhiều ô từ Excel và nhấn Ctrl+V tại ô bắt đầu của WorkGrid, hệ thống tự động dải đè mảng dữ liệu tương ứng vào lưới.
         * *Chế độ Điền nhanh (Single-value fill)*: Khi chỉ copy 1 ô số lượng giờ (ví dụ copy số `8.0` hoặc `0.0`) và bôi đen cả một vùng lớn trên WorkGrid rồi nhấn Ctrl+V, phần mềm tự động điền giá trị đó cho tất cả các ô đang được chọn.
      3. **Tối ưu hóa thuật toán CPM tôn trọng số 0 thủ công**: Trước đây, thuật toán CPM tự động phân bổ tài nguyên luôn tự ý ghi đè số `0` do kỹ sư nhập thủ công thành `8.0` ở các ngày làm việc thường. Phiên bản v0.2.0-01 đã sửa lỗi bảo thủ này bằng cách loại bỏ điều kiện ghi đè tại hàm `SyncAssignmentsWithCalendar()`. Giờ đây, hệ thống tuyệt đối tôn trọng các ô có giá trị `0` do người dùng dán hoặc sửa thủ công, giúp phản ánh chính xác kế hoạch bố trí nhân công/máy móc thi công ngắt quãng.
      4. **Giao diện Ribbon tinh gọn**: Ẩn (Collapsed) các nút Xuất/Nhập CSV cũ trên bảng chấm công để hướng người dùng sang workflow Copy-Paste trực tiếp từ Excel siêu tốc và không phát sinh lỗi tệp tin.
