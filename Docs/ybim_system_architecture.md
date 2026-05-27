# SƠ ĐỒ KIẾN TRÚC VÀ LƯU LƯỢNG DỮ LIỆU: YBIM TDTC v0.2.0

Được cập nhật tự động thay thế cho hình ảnh PNG tĩnh cũ.

``mermaid
graph TD
    %% Tùy chỉnh màu sắc
    classDef wpf fill:#1E3A8A,stroke:#3B82F6,stroke-width:2px,color:#fff,rx:10,ry:10;
    classDef core fill:#0F172A,stroke:#10B981,stroke-width:2px,color:#fff,rx:10,ry:10;
    classDef engine fill:#312E81,stroke:#8B5CF6,stroke-width:2px,color:#fff,rx:10,ry:10;
    classDef data fill:#064E3B,stroke:#059669,stroke-width:2px,color:#fff,rx:10,ry:10;
    classDef export fill:#7F1D1D,stroke:#EF4444,stroke-width:2px,color:#fff,rx:10,ry:10;

    subgraph UI ["LỚP GIAO DIỆN NGƯỜI DÙNG (WPF FLUENT)"]
        Ribbon["Thanh Ribbon đa chức năng"]:::wpf
        Theme["Chế độ Sáng / Tối"]:::wpf
        GanttView["Gantt Chart Window"]:::wpf
        ResourceView["Dashboard Tài Nguyên"]:::wpf
    end

    subgraph INPUT ["TRÌNH NHẬP EXCEL SONG NGỮ"]
        Fuzzy["Khớp cột Fuzzy Logic"]:::core
        LinkParser["Giải mã liên kết (SS, FS)"]:::core
        InputExcel[("Dữ liệu gốc\n(.xlsx)")]:::data
    end

    subgraph ENGINE ["ĐỘNG CƠ LÕI (CORE ENGINES)"]
        CPM["Thuật toán Tính toán CPM\n(Đường găng, Topo Sort)"]:::engine
        AI["Tối ưu Di truyền (AI)\n(San bằng tài nguyên)"]:::engine
        Skia["SkiaSharp Graphics\n(Render 3 Pha, Cố định trục)"]:::engine
        VC["Quản lý Phiên bản\n(Undo/Redo & Snapshot)"]:::engine
    end

    subgraph STORAGE ["LƯU TRỮ TRẠNG THÁI"]
        JSON[("System.Text.Json\n(.ybim file)")]:::data
        RAM[("Undo/Redo Stack\n(Memory)")]:::data
    end

    subgraph EXPORT ["LỚP XUẤT BẢN & BÁO CÁO"]
        PDF["Xuất Báo cáo PDF\n(QuestPDF)"]:::export
        ExcelOut["Xuất Dự án Excel\n(C# ⇌ Python)"]:::export
        HTML["Dashboard HTML\n(Chart.js)"]:::export
        LISP["Xuất bản vẽ CAD\n(AutoCAD Lisp)"]:::export
    end

    %% Các luồng kết nối
    InputExcel -->|Nhập dữ liệu| Fuzzy
    Fuzzy --> LinkParser
    LinkParser -->|Dữ liệu thô| CPM

    Ribbon -->|Điều khiển| CPM
    Ribbon -->|Điều khiển| AI
    Ribbon -->|Yêu cầu kết xuất| EXPORT

    CPM <-->|Tính toán & Lịch trình| JSON
    AI -->|3 Phương án tối ưu| JSON
    VC <-->|Lưu nháp / Khôi phục| JSON
    JSON -->|Dữ liệu thời gian thực| RAM

    JSON -->|Nguồn dữ liệu| Skia
    Skia -->|Vẽ đồ họa tối ưu| GanttView
    Skia -->|Vẽ đồ thị| ResourceView

    JSON -->|Xuất bản| PDF
    JSON -->|Xuất bản| ExcelOut
    JSON -->|Xuất bản| HTML
    JSON -->|Xuất bản| LISP
``
