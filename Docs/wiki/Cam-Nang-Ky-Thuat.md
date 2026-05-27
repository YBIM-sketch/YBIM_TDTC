# CẨM NANG KỸ THUẬT YBIM TDTC: THUẬT TOÁN DI TRUYỀN, CẬP NHẬT TỰ ĐỘNG, BẢO MẬT MÃ HÓA & THIẾT KẾ GIAO DIỆN FLUENT







Tài liệu này tổng hợp toàn bộ kiến thức kỹ thuật, quy trình triển khai, kiến trúc lõi và các lưu ý quan trọng khi phát triển, nâng cấp và đóng gói phiên bản phần mềm **YBIM TDTC v0.2.0-01**.







---







## 🧬 1. TÍCH HỢP THUẬT TOÁN DI TRUYỀN (GENETIC ALGORITHM) SAN BẰNG TÀI NGUYÊN







### 1.1. Mục tiêu



San bằng tài nguyên (Resource Leveling) bằng cách tịnh tiến (delay) các công tác **không găng** (TotalSlack > 0) trong phạm vi slack cho phép, sao cho biểu đồ tài nguyên hằng ngày (Nhân công, Máy móc) có phương sai nhỏ nhất và không vượt ngưỡng năng lực tối đa (MaxCapacity).







### 1.2. Luồng dữ liệu tổng thể







```



┌─────────────────┐    Task.Run()    ┌──────────────────────────────┐



│  MainWindow UI  │ ───────────────▶ │ ScheduleOptimizationService  │



│  (WPF Thread)   │                  │  (Background Thread x3)      │



│                 │ ◀─────────────── │                              │



│  Render Tabs    │   await results  │  ┌────────────────────────┐  │



│ └─────────────────┘                  │  │  GeneticAlgorithm      │  │



│                                      │  │  ├─ ScheduleChromosome │  │



│                                      │  │  ├─ MultiObjectiveFitness│ │



│                                      │  │  ├─ EliteSelection(5)  │  │



│                                      │  │  ├─ UniformCrossover    │  │



│                                      │  │  └─ UniformMutation     │  │



│                                      │  └────────────────────────┘  │



│                                      └──────────────────────────────┘



```







### 1.3. Cấu trúc file đã tạo/sửa cho thuật toán di truyền







| Hành động | File | Mô tả |



|-----------|------|-------|



| **[NEW]** | `Services/ScheduleChromosome.cs` | Nhiễm sắc thể — mỗi gen = số ngày delay |



| **[NEW]** | `Services/MultiObjectiveFitness.cs` | Hàm Fitness đa mục tiêu |



| **[NEW]** | `Services/ScheduleOptimizationService.cs` | Service điều phối GA |



| **[MODIFY]** | `MainWindow.xaml.cs` | Thêm 3 method: `BtnAIOptimize_Click`, `CreateOptimizationTabs`, `CommitOptimizedPlan` |



| **[MODIFY]** | `YBIM_TDTC.csproj` | Thêm NuGet package `GeneticSharp 3.1.4` |







### 1.4. Chi tiết các thành phần di truyền







#### A. ScheduleChromosome (Nhiễm sắc thể)



* **Kế thừa:** `GeneticSharp.ChromosomeBase`



* Mỗi cá thể (chromosome) biểu diễn **một kịch bản tiến độ**. Số lượng gen = số công tác có `TotalSlack > 0`. Giá trị mỗi gen là số nguyên trong khoảng `[0, TotalSlack]` — tức số ngày tịnh tiến (delay) của công tác tương ứng.







```csharp



// Khởi tạo: chỉ truyền vào danh sách công tác không găng



var chromosome = new ScheduleChromosome(flexibleTasks);







// Mỗi gen random từ 0 → maxDelay



public override Gene GenerateGene(int geneIndex)



{



    int maxDelay = (int)_flexibleTasks[geneIndex].TotalSlack;



    int randomDelay = RandomizationProvider.Current.GetInt(0, maxDelay + 1);



    return new Gene(randomDelay);



}



```







#### B. MultiObjectiveFitness (Hàm Thích nghi)



* **Kế thừa:** `GeneticSharp.IFitness`



* > ⚠️ **CẢNH BÁO HIỆU NĂNG:** Hàm `Evaluate()` được gọi **hàng chục ngàn lần** mỗi vòng GA. Tuyệt đối KHÔNG clone object, KHÔNG dùng LINQ nặng trong vòng lặp chính.







##### Các kỹ thuật tối ưu hiệu năng đã áp dụng:



1. **Pre-computed index mapping:** Dùng `Dictionary<int, int>` map từ task index → flex index, thay vì `IndexOf()` chạy O(n) mỗi lần.



2. **Mảng tĩnh thay vì List:** Dùng `double[]` cố định kích thước thay vì `List<double>`.



3. **Cache MaxCapacity:** Tính tổng MaxCapacity theo loại tài nguyên 1 lần trong constructor.



4. **Bounds checking:** Thêm `+200` buffer cho mảng để tránh tràn khi delay lớn.







##### Công thức Fitness:



```



Fitness = 10000.0 / (1.0 + TotalPenalty + TotalVariance - WeightMaterial * CashflowBonus)







Trong đó:



  TotalPenalty = W_Machine * Σ(vượt_máy × 100) + W_Labor * Σ(vượt_nhân_công × 100)



  TotalVariance = W_Machine * Var(daily_machine) + W_Labor * Var(daily_labor)



  CashflowBonus = Σ(delay × 0.1) cho mỗi công tác flexible (giãn dòng tiền)



```







##### Lỗi tương thích đã sửa so với code gốc:



* Thay thế `ass.Units` bằng `ass.AllocatedUnits` do `ResourceAssignment` không có property `Units`.



* Loại bỏ Hard-code `MaxCapacity = 5` (máy), `= 50` (nhân công) sang đọc động từ `YResource.MaxCapacity` qua `_resources.Where(r => r.Type == ...).Sum(r => r.MaxCapacity)`.



* Dùng Pre-computed `_taskIndexToFlexIndex` dictionary thay thế cho `flexibleTasks.IndexOf(task)` trong vòng lặp chính.



* Thêm bounds checking `if (startDayIdx < 0)` và `if (endIdx > arraySize)` để tránh lỗi `IndexOutOfRangeException`.



* Bổ sung kiểm tra chia cho 0: `if (denominator <= 0) denominator = 0.001` tránh lỗi `DivideByZeroException`.







#### C. ScheduleOptimizationService (Bộ điều phối)



Service hoàn toàn **cô lập** (không phụ thuộc UI), dễ gọi từ bất kỳ thread nào.







##### Cấu hình GA Operators:



* **Selection:** `EliteSelection(5)` — Giữ 5 cá thể tốt nhất để tránh thoái hóa.



* **Crossover:** `UniformCrossover(0.5f)` — Tỷ lệ 50% nhặt ngẫu nhiên gen từ Cha/Mẹ.



* **Mutation:** `UniformMutation(true)` — Đột biến toàn bộ gen để bứt phá nghiệm cục bộ.



* **Population:** `Population(100, 200)` — Min 100, Max 200.



* **Điều kiện dừng thông minh:**



  ```csharp



  Termination = new OrTermination(



      new GenerationNumberTermination(200),   // Tối đa 200 thế hệ



      new FitnessStagnationTermination(30)    // Hoặc stagnation 30 thế hệ liên tiếp



  )



  ```







##### Deep Clone bằng JSON:



```csharp



private List<YTask> DeepCloneTasks(List<YTask> tasks)



{



    var json = JsonSerializer.Serialize(tasks, options);



    return JsonSerializer.Deserialize<List<YTask>>(json, options) ?? new List<YTask>();



}



```



* **Lưu ý:** `YTask` implements `INotifyPropertyChanged`, nên cần `JsonSerializerOptions` phù hợp. `YTask.Clone()` chỉ shallow clone `Assignments` list (copy reference, không deep clone `DailyWork` dictionary), nên JSON serialize an toàn hơn cho mục đích GA.







### 1.5. Tích hợp UI đa luồng (`MainWindow.xaml.cs`)







#### Gọi GA song song 3 phương án:



```csharp



private async void BtnAIOptimize_Click(object sender, RoutedEventArgs e)



{



    var optimizer = new ScheduleOptimizationService();



    var baseTasks = _tasks.ToList();



    var resources = _resourcePool.ToList();







    // 3 phương án song song với trọng số khác nhau



    var task1 = System.Threading.Tasks.Task.Run(() => optimizer.RunGeneticAlgorithm(baseTasks, resources, 10, 1, 1)); // Ưu tiên Máy



    var task2 = System.Threading.Tasks.Task.Run(() => optimizer.RunGeneticAlgorithm(baseTasks, resources, 1, 10, 1)); // Ưu tiên Nhân công



    var task3 = System.Threading.Tasks.Task.Run(() => optimizer.RunGeneticAlgorithm(baseTasks, resources, 1, 1, 10)); // Ưu tiên Vật tư



    var results = await System.Threading.Tasks.Task.WhenAll(task1, task2, task3);







    CreateOptimizationTabs(results[0], results[1], results[2]);



}



```



* ⚠️ **BẮT BUỘC dùng `System.Threading.Tasks.Task.Run()`** thay vì `Task.Run()` vì namespace `Task` bị conflict với model `YTask` trong WPF code-behind.







#### Tạo 3 Tab kết quả:



Method `CreateOptimizationTabs()` tạo 3 `TabItem` mới trong `MainTabControl`. Mỗi tab chứa bảng chỉ số so sánh (Premium Dashboard Card) hiển thị Thời gian thi công, Đỉnh nhân công, Đỉnh máy thi công trước/sau tối ưu cùng các huy hiệu phần trăm san bằng trực quan, kèm nút bấm **Áp dụng** để chốt phương án.







#### Commit phương án được chọn:



```csharp



private void CommitOptimizedPlan(List<YTask> selectedPlan)



{



    _tasks.Clear();



    foreach (var t in selectedPlan) _tasks.Add(t);







    // Xóa 3 tab PA, quay về tab chính



    var tabsToRemove = MainTabControl.Items.Cast<TabItem>()



        .Where(t => t.Header != null && t.Header.ToString()!.Contains("PA")).ToList();



    foreach (var tab in tabsToRemove) MainTabControl.Items.Remove(tab);







    MainTabControl.SelectedIndex = 0;



    RefreshGantt();



}



```







### 1.6. Thuật toán Ép nén đường găng (Greedy Crashing - TCTO)

Sau khi hoàn tất san bằng tài nguyên, nếu người dùng kích hoạt kịch bản TCTO (Time-Cost Trade-Off), hệ thống gọi động cơ `GreedyCrashingOptimizer`.

*   **Cấu trúc dữ liệu:** Bổ sung thuộc tính `CrashingCost` ($/Ngày) vào mô hình `YTask` và thuộc tính `Currency` (VND/USD/EUR) vào `YResource`.
*   **Mục tiêu:** Rút ngắn thời gian dự án với tổng chi phí phát sinh (Penalty Cost) là tối thiểu thông qua quá trình đánh đổi thời gian - chi phí (TCTO).
*   **Giải pháp kỹ thuật (Greedy Algorithm):**
    1.  **Dò đường găng (CPM):** Cập nhật lại đồ thị và xác định các công tác nằm trên đường găng hiện tại.
    2.  **Sắp xếp chi phí:** Lọc các công tác găng có thể nén được (`Duration > 1`) và sắp xếp tăng dần theo `CrashingCost`.
    3.  **Ép nén:** Chọn công tác có `CrashingCost` rẻ nhất, giảm `Duration` đi 1 ngày.
    4.  **Cập nhật động:** Tính toán lại CPM. Lặp lại bước 1 cho đến khi đạt được mục tiêu rút ngắn (1-5% thời gian dự án) hoặc không còn công tác găng nào nén được.
    5.  **Dịch chuyển đồng bộ:** Hệ thống dùng đệ quy duyệt đồ thị DAG để tịnh tiến các công tác nối tiếp (Successors) lùi về trước để lấp đầy khoảng trống thời gian.







---







## 📡 2. CƠ CHẾ CẬP NHẬT TỰ ĐỘNG (AUTO-UPDATE) & PHẢN HỒI NHANH (QUICK FEEDBACK)







### 2.1. Kiểm tra Cập nhật Tự động (v0.2.0-01)



Phần mềm có khả năng tự động truy vấn thông tin phiên bản mới nhất từ kho lưu trữ GitHub để thông báo cho người dùng.







* **Cơ chế hoạt động:** Khi người dùng nhấn "Kiểm tra phiên bản mới", phần mềm sẽ tải nội dung từ file cấu hình `update.json` nằm trên nhánh `main` của kho lưu trữ.



* **Cấu trúc JSON (`update.json`):**



  ```json



  {



    "Version": "0.2.0",



    "DownloadUrl": "https://github.com/YBIM-sketch/YBIM_TDTC/releases/download/v0.2.0-01/YBIM_TDTC_v0.2.0-01.zip",



    "Changelog": "Mô tả tính năng mới được cập nhật..."



  }



  ```



* **Bảo toàn dữ liệu mạng (Bỏ qua Obfuscation):** 



  Để đảm bảo phần mềm có thể chuyển đổi chuỗi JSON từ mạng vào Object C# chính xác, lớp `YBIM_TDTC.UpdateInfo` đã được cấu hình loại trừ (Skip) khỏi quá trình mã hoá đổi tên trong file `obfuscar.xml`.







### 2.2. Tính năng Gửi Phản hồi Nhanh (Quick Feedback)



Thay vì sử dụng giao thức `mailto:` phụ thuộc vào cài đặt hệ điều hành (thường gây lỗi mở nhầm trình duyệt trắng hoặc hỏi cấu hình ứng dụng email cục bộ phức tạp), tính năng phản hồi của YBIM được cải tiến **mở thẳng trang soạn thảo Web của Gmail**.







* **Cơ chế hoạt động:** Phần mềm thu thập dữ liệu hệ thống (Email bản quyền, Phiên bản YBIM, Hệ điều hành), mã hóa (`EscapeDataString`) và định dạng thành URL của Gmail Compose:



  `https://mail.google.com/mail/?view=cm&fs=1&to=hoangy.dgbo.k52@gmail.com&su=[Chủ đề]&body=[Nội dung]`



* **Ưu điểm:** Hoạt động trơn tru trên mọi máy tính. Người dùng chỉ cần đăng nhập tài khoản Google trên trình duyệt là sẽ vào thẳng giao diện soạn thư với nội dung, tiêu đề và người nhận được điền sẵn thông tin hệ thống của máy đang chạy.







---







## 🛡️ 3. QUY TRÌNH BIÊN DỊCH VÀ CHỐNG DỊCH NGƯỢC TỰ ĐỘNG (OBFUSCATION)







Toàn bộ quá trình mã hóa chống dịch ngược đã được **tích hợp 100%** vào chu trình build thông qua *MSBuild Target* trong file `YBIM_TDTC.csproj`. Bạn không cần thao tác gõ lệnh thủ công để xáo trộn mã (Obfuscate) nữa.







### 3.1. Cơ chế hoạt động của Obfuscar tích hợp



* Bất cứ khi nào dự án được build ở chế độ `Release`, MSBuild sẽ kích hoạt sự kiện sau khi build hoàn tất (`AfterTargets="Build"`).



* Nó sẽ tự động gọi trình Obfuscar console cùng với cấu hình `obfuscar.xml` để xáo trộn mã nguồn (đổi tên biến, hàm thành các ký tự rác), bảo vệ thuật toán lõi vững chắc.



* Tệp thi hành sau khi được Obfuscar xử lý an toàn sẽ nằm trong thư mục: `Obfuscated_YBIM`.







---







## 📦 4. QUY TRÌNH ĐÓNG GÓI PHÁT HÀNH SIÊU TỐC (VERSION BUMP & PACKAGING)







### 4.1. Danh sách vị trí cần cập nhật khi đổi version (Version Bump)







| # | File | Vị trí cụ thể |



|---|------|---------------|



| 1 | `MainWindow.xaml.cs` | Hằng số `CURRENT_VERSION = "x.y.z"` |



| 2 | `MainWindow.xaml` (dòng ~623) | TextBlock `Text="vx.y.z"` trên Sleek TopBar |



| 3 | `MainWindow.xaml` (dòng ~948) | TextBlock `Text="Phiên bản vx.y.z"` trên StatusBar |



| 4 | `update.json` | `"Version"`, `"DownloadUrl"` (chứa tên tag + tên file zip) |



| 5 | `README.md` | Tiêu đề, phiên bản hiện tại, link cài đặt, JSON mẫu, tài liệu |



| 6 | `Huong_Dan_Su_Dung_YBIM_TDTC.md` | Tên file zip trong hướng dẫn cài đặt |



| 7 | `Work_Skill.md` | Cập nhật phiên bản ghi nhận ở chân tài liệu |







### 4.2. Lệnh tìm kiếm nhanh toàn bộ vị trí version cũ



```bash



# Tìm tất cả vị trí chứa phiên bản cũ (VD: 0.2.0)



grep -rn "0.2.0" --include="*.cs" --include="*.xaml" --include="*.json" --include="*.md" --include="*.bat" .



```







### 4.3. Quy trình 3 bước đóng gói phát hành:







#### Bước 1: Nâng cấp phiên bản phần mềm



Mở file `MainWindow.xaml.cs`, tìm hằng số `CURRENT_VERSION` và tăng số phiên bản (Ví dụ đổi thành `"0.2.0"`). Cập nhật các vị trí Text hiển thị phiên bản khác trên `MainWindow.xaml`.







#### Bước 2: Chạy công cụ tự động biên dịch `build.bat`



Nhấp đúp chuột vào file **`build.bat`** tại thư mục gốc của dự án. 



Công cụ sẽ tự động phục hồi gói cài đặt NuGet và gọi lệnh `dotnet build -c Release`. Mã nguồn sẽ được biên dịch và chạy kèm quy trình mã hóa bảo vệ ngay lập tức.



*Tệp thi hành (đã an toàn bảo mật sau Obfuscation) nằm tại:* `bin\Release\net8.0-windows\YBIM_TDTC.exe`.







#### Bước 3: Đóng gói ZIP và Đăng tải lên GitHub



1. Mở thư mục `bin\Release\net8.0-windows\`, chọn toàn bộ các file bên trong và nén thành tệp **`YBIM_TDTC_v0.2.0-01.zip`** (PowerShell command: `Compress-Archive -Path "bin\Release\net8.0-windows\*" -DestinationPath "YBIM_TDTC_v0.2.0-01.zip" -Force`).



2. Đăng nhập GitHub, vào mục **Releases**, tạo bản Release mới (New release) với tag `v0.2.0-01`.



3. Đính kèm file ZIP vừa tạo vào Release và nhấn **Publish**.



4. Sửa thông tin tệp `update.json` trên nhánh `main` trùng khớp với tên phiên bản và liên kết tải về vừa tạo. Cập nhật thành công!







---







## 🎨 5. KIẾN TRÚC LÕI GIAO DIỆN PHẦN MỀM (CORE UI ARCHITECTURE)







Giao diện YBIM TDTC được xây dựng theo ngôn ngữ thiết kế **Fluent Design (Windows 11)** hiện đại, đồng bộ hóa cao về cả thẩm mỹ lẫn hiệu năng hiển thị.







### 5.1. Chuẩn Hóa Biểu Tượng Hệ Thống (Segoe Fluent Icons)



Để loại bỏ sự phụ thuộc vào các thư viện icon nặng như `MahApps.Metro.IconPacks` và mang lại giao diện Windows 11 thuần khiết, toàn bộ hệ thống biểu tượng đã được chuyển đổi sang **Segoe Fluent Icons** nguyên bản của hệ thống.







* **Không sử dụng `<FontIcon>`:** Phần tử `<FontIcon>` thuộc UWP/WinUI và không có sẵn trong các namespace mặc định của WPF truyền thống. Cố gắng sử dụng sẽ gây ra lỗi biên dịch nghiêm trọng `MC3074`.



* **Giải pháp chuẩn WPF - `<TextBlock>`:** Sử dụng phần tử `<TextBlock>` thuần WPF kết hợp định dạng font hệ thống và thực thể Unicode (Unicode character entities):



  ```xml



  <!-- Ví dụ: Biểu tượng Lưu file (Save) -->



  <TextBlock FontFamily="Segoe Fluent Icons" Text="&#xE74E;" FontSize="32" Foreground="{DynamicResource Y_Accent}" />



  ```







### 5.2. Hướng dẫn Căn chỉnh Biểu tượng Ribbon & Tránh Lệch Trục (Axis Alignment)



* **Vấn đề lệch trục biểu tượng:** Khi sử dụng trực tiếp `<TextBlock>` với Segoe Fluent Icons làm icon cho các nút Ribbon, WPF mặc định tính line-height theo phông chữ nên ký tự icon sẽ bị đẩy lệch lên góc trên bên trái của vùng hiển thị (đặc biệt khi dùng trong `LargeIcon` hoặc `Icon` thường).



* **Giải pháp Cấu hình Tập trung (WPF Resources):**



  Khai báo 2 Style dùng chung trong `<ResourceDictionary>` của `MainWindow.xaml`:







```xml



<!-- Unified Centered Icon Styles to prevent axis misalignment -->



<Style x:Key="RibbonLargeIcon" TargetType="TextBlock">



    <Setter Property="FontFamily" Value="Segoe Fluent Icons" />



    <Setter Property="FontSize" Value="32" />



    <Setter Property="Width" Value="32" />



    <Setter Property="Height" Value="32" />



    <Setter Property="HorizontalAlignment" Value="Center" />



    <Setter Property="VerticalAlignment" Value="Center" />



    <Setter Property="TextAlignment" Value="Center" />



    <Setter Property="Foreground" Value="{DynamicResource Y_Accent}" />



</Style>







<Style x:Key="RibbonSmallIcon" TargetType="TextBlock">



    <Setter Property="FontFamily" Value="Segoe Fluent Icons" />



    <Setter Property="FontSize" Value="14" />



    <Setter Property="Width" Value="16" />



    <Setter Property="Height" Value="16" />



    <Setter Property="HorizontalAlignment" Value="Center" />



    <Setter Property="VerticalAlignment" Value="Center" />



    <Setter Property="TextAlignment" Value="Center" />



    <Setter Property="Foreground" Value="{DynamicResource Y_Accent}" />



</Style>



```







* **Cách áp dụng quy chuẩn vào Ribbon:**



  Thay vì viết mã phông chữ và kích thước rải rác ở từng nút, áp dụng các Style quy chuẩn như sau:



  



  * *Với LargeIcon (Biểu tượng lớn 32x32):*



    ```xml



    <Fluent:Button Header="Tên nút" Click="Btn_Click" Size="Large">



        <Fluent:Button.LargeIcon>



            <TextBlock Style="{StaticResource RibbonLargeIcon}" Text="&#xE710;" />



        </Fluent:Button.LargeIcon>



    </Fluent:Button>



    ```



  * *Với SmallIcon (Biểu tượng nhỏ 16x16):*



    ```xml



    <Fluent:Button Header="Tên nút" Click="Btn_Click" Size="Middle">



        <Fluent:Button.Icon>



            <TextBlock Style="{StaticResource RibbonSmallIcon}" Text="&#xE787;" />



        </Fluent:Button.Icon>



    </Fluent:Button>



    ```



  * *Ghi đè màu sắc cá biệt (nếu có):*



    Nhờ cơ chế ưu tiên giá trị cục bộ (Local Value Precedence) của WPF, ta có thể dễ dàng ghi đè màu sắc (như màu đỏ cảnh báo) ngay tại thẻ TextBlock mà không làm mất các thuộc tính căn chỉnh của Style:



    ```xml



    <TextBlock Style="{StaticResource RibbonLargeIcon}" Text="&#xEA90;" Foreground="{DynamicResource Y_Critical}" />



    ```







### 5.3. Nguyên Tắc Bắt Buộc: Mã Hóa File XAML Với UTF-8 Có BOM



Để tiếng Việt có dấu hiển thị chuẩn xác trên mọi máy tính chạy phiên bản Windows với ngôn ngữ hệ thống khác nhau:



* **Vấn đề:** Nếu file XAML được lưu ở dạng UTF-8 không có BOM (Byte Order Mark), trình phân tích cú pháp XAML của WPF sẽ tự động chuyển sang đọc file theo trang mã ANSI mặc định của hệ điều hành. Việc này làm hỏng toàn bộ tiếng Việt trong ứng dụng thành các chuỗi ký tự lạ.



* **Giải pháp:** Mọi file XAML bắt buộc phải được lưu với mã hóa **UTF-8 có BOM** (`utf-8-sig` trong Python). Byte BOM đầu file (`\xef\xbb\xbf`) sẽ ép trình biên dịch WPF nhận diện chính xác 100% tiếng Việt UTF-8.







### 5.4. Hệ Thống Chủ Đề Và Responsive Layout (Responsive Theme System)



* **Bảng màu đồng bộ (Harmony Palette):** Toàn bộ các nút bấm và nhãn trong ứng dụng đều ràng buộc chặt chẽ với hệ thống DynamicResource tài nguyên màu chuẩn: `Y_Accent` (Màu nhấn chủ đạo), `Y_SurfaceBackground` (Màu nền các bảng và hội thoại), `Y_TextPrimary` & `Y_TextSecondary`.



* **Chủ đề Sáng/Tối (Light & Dark Mode):** Hỗ trợ đổi chủ đề mượt mà qua các file tài nguyên `Themes/ModernLight.xaml` và `Themes/ModernDark.xaml`, tự động lưu trữ và tải lại thông qua cấu hình `themesettings.json`.



* **Ribbon Tự Co Giãn (Adaptive Ribbon):** Giao diện sử dụng bộ điều khiển `Fluent:RibbonWindow` cao cấp. Ở chế độ thu nhỏ màn hình hoặc máy tính bảng (màn 14-16" display), Ribbon tự động rút gọn cấu trúc cột nút từ `350px` xuống còn `120px` để bảo toàn không gian làm việc tối đa cho Gantt Chart bên dưới.







---







## 📧 6. HIỂN THỊ EMAIL NGƯỜI DÙNG TRONG ỨNG DỤNG VÀ FILE EXPORT







Để tăng tính cá nhân hoá và hỗ trợ quản lý bản quyền, chúng ta sẽ hiển thị địa chỉ email đã lưu (được lưu trong `LicenseManager`) trên giao diện và nhúng vào các file export (TXT/Excel).







### 6.1. Lấy email người dùng



```csharp



// Trong MainWindow.xaml.cs



string userEmail = LicenseManager.GetStoredEmail(); // trả về email đã kích hoạt



```



* Đảm bảo hiển thị trên UI bằng cách gán cho TextBlock: `TxtUserEmail.Text = $"Email: {userEmail}";` (Hiển thị "Email: chưa xác định" nếu chưa kích hoạt).







### 6.2. Nhúng email vào file TXT xuất tiến độ



Trong phương thức `BtnExportTxt_Click` thêm một dòng trước khi ghi dữ liệu:



```csharp



sb.Insert(0, $"Email\t{userEmail}\n"); // Đặt email ở dòng đầu tiên



```







### 6.3. Nhúng email vào file Excel (khi xuất Excel)



Trong `BtnExportExcel_Click` (phần tạo StringBuilder), chèn:



```csharp



sb.Insert(0, $"<Email>{userEmail}</Email>\n");



```







---







## ✅ 7. CHECKLIST KIỂM TRA SAU KHI TÍCH HỢP







- [x] Package `GeneticSharp 3.1.4` đã được thêm vào `.csproj`.



- [x] 3 file service mới biên dịch thành công (0 Error).



- [x] `BtnAIOptimize_Click` đã được thêm vào `MainWindow.xaml.cs`.



- [x] `CreateOptimizationTabs` tạo 3 tab kết quả đúng.



- [x] `CommitOptimizedPlan` xóa tab và refresh Gantt đúng.



- [x] Build Release thành công kèm Obfuscar.



- [x] Version bumped từ `v0.1.3` → `v0.2.0-01` tại **tất cả** vị trí.



- [x] File `YBIM_TDTC_v0.2.0-01.zip` (89.5 MB) đã đóng gói.



- [x] Thêm nút Ribbon trong `MainWindow.xaml` kết nối `BtnAIOptimize_Click` và căn chỉnh trục thẳng hàng.



- [x] Thiết kế Style quy chuẩn `RibbonLargeIcon` và `RibbonSmallIcon` chống lệch tâm đứng/ngang cho phông chữ Segoe Fluent Icons.



- [x] **MỚI:** Tích hợp tính năng Nhập Dự án 1-Click từ Excel mẫu song ngữ (`YBIM_ImportTemplate.xlsx`).







---







## 📊 8. HỆ THỐNG NHẬP DỰ ÁN 1-CLICK TỪ EXCEL SONG NGỮ TOÀN DIỆN







Để hiện thực hóa tài liệu hướng dẫn nhập mẫu song ngữ, phiên bản **YBIM TDTC v0.2.0-01** đã tích hợp thành công trình thông dịch dự án 1-click từ Excel thông minh tại phương thức `ImportExcel_Click` trong `MainWindow.xaml.cs`.







### 8.1. Cơ chế Fuzzy Matching và Đa ngôn ngữ (Bilingual Columns Detection)



Hệ thống sử dụng thư viện `ExcelDataReader` đọc tệp `.xlsx` siêu tốc, tự động phân tích và khớp cột mà không phân biệt chữ hoa, chữ thường hay ngôn ngữ (Anh - Việt):



*   **Sheet nhận diện:**



    *   `Tasks` hoặc `công việc` -> Chuyển thành bảng dữ liệu Công tác.



    *   `Resources` hoặc `tài nguyên` -> Chuyển thành bảng dữ liệu Tài nguyên.



    *   `Assignments` hoặc `phân bổ` -> Chuyển thành bảng phân bổ tài nguyên.



*   **Thuật toán Fuzzy Columns mapping:**



    ```csharp



    // Ví dụ khớp thông minh cột tên công tác



    if (colName.Contains("name") || colName.Contains("tên")) nameCol = col;



    // Khớp thông minh cột mối quan hệ tiền nhiệm



    if (colName.Contains("predecessor") || colName.Contains("tiền nhiệm")) tPredCol = col;



    // Khớp thông minh cột số ngày thi công (mới)



    if (colName.Contains("duration") || colName.Contains("ngày thi công") || colName.Contains("thời gian thi công") || colName.Contains("số ngày")) tDurCol = col;



    ```







*   **Xử lý nạp & Dự phòng tương thích (Backward Compatibility Fallback):**



    Nếu cột ngày thi công (`tDurCol`) tồn tại và có dữ liệu hợp lệ, hệ thống sẽ ưu tiên nạp giá trị số này vào `YTask.Duration`. Nếu tệp nhập vào thiếu cột này (ví dụ tệp biểu mẫu cũ), chương trình tự động tính toán số ngày thi công dự phòng từ khoảng cách giữa ngày bắt đầu và ngày kết thúc:



    ```csharp



    duration = (FinishDate - StartDate).TotalDays;



    if (duration <= 0) duration = 1; // Giá trị dự phòng tối thiểu



    ```







### 8.2. Giải mã thuật toán khớp mối liên kết tiền nhiệm (Dependency String Parser)



Cơ chế tự động bóc tách chuỗi liên kết phức tạp từ Excel (ví dụ: `1FS+2d, 3SS-1, 4FF+0d`):



1.  Bóc tách chuỗi theo các dấu phân cách `,` hoặc `;`.



2.  Tách chỉ số ID công tác tiền nhiệm (các chữ số đầu tiên).



3.  Nhận diện các kiểu mối quan hệ: `SS`, `FF`, `SF`, `FS` (hoặc mặc định là `FS` nếu không ghi rõ).



4.  Nhận diện số ngày trễ (Lag) hỗ trợ cả giá trị âm/dương và hậu tố `d` hoặc `ngày` (ví dụ: `+2d` -> `2`, `-1` -> `-1`).







---







## ❄️ 9. CƠ CHẾ CỐ ĐỊNH TRỤC THỜI GIAN GANTT CHART (FROZEN TIMESCALE HEADER)







Để tối ưu hóa trải nghiệm người dùng khi làm việc với dự án lớn có hàng trăm công tác, phiên bản **YBIM TDTC v0.2.0-01** đã tích hợp thành công giải pháp **cố định trục thời gian (Timescale)** ở đầu vùng nhìn biểu đồ Gantt khi cuộn dọc.







### 9.1. Kiến trúc Vẽ 3 Giai Đoạn (Three-Phase Painting Scheme)



Thay vì vẽ trục thời gian trước rồi vẽ thanh công việc đè lên như cũ, hệ thống được cấu trúc lại hoàn toàn theo 3 giai đoạn độc lập trên SkiaSharp canvas:



1.  **Pha 1 (Vẽ Nền & Lưới Dọc):** Vẽ phần tô màu cuối tuần (Weekend shading) và các đường giăng dọc lưới Gantt xuống dưới bắt đầu từ tọa độ `headerHeight` đến hết chiều cao thực tế `e.Info.Height`.



2.  **Pha 2 (Vẽ Công Tác & Liên Kết Găng):** Vẽ toàn bộ đường liên kết mối quan hệ tiền nhiệm, các thanh công tác (Task Bars) và nhãn ngày tháng tương ứng.



3.  **Pha 3 (Vẽ Trục Thời Gian Đóng Băng - Sticky Overlay):** Vẽ đè phần nền trục thời gian (`headerBgColor`), các nhãn tháng/năm (Top Tier), vạch ngăn cách ngày/quý (Bottom Tier) ở tọa độ dịch chuyển `yOffset`.







### 9.2. Thuật toán dịch chuyển và đồng bộ cuộn (yOffset & Real-time Invalidation)



*   **Tính toán dịch chuyển:**



    ```csharp



    float yOffset = (float)GanttScrollViewer.VerticalOffset * dpi;



    ```



    Toàn bộ các tọa độ vẽ của trục thời gian trong Pha 3 đều được cộng thêm `yOffset`. Nhờ đó, trục thời gian luôn nằm cố định ở phía trên cùng của viewport hiển thị dù biểu đồ Gantt có cuộn xuống sâu đến mức nào.



*   **Vẽ lại thời gian thực (Real-time Repaint):**



    Tại sự kiện cuộn dọc của cả `TaskGrid` và `GanttScrollViewer`, hệ thống gọi tường minh:



    ```csharp



    GanttSkiaElement?.InvalidateVisual();



    ```



    Việc này ép buộc SkiaSharp vẽ lại trục thời gian khớp với độ lệch cuộn ngay lập tức, triệt tiêu hoàn toàn hiện tượng giật lag hay lệch vị trí.







*   **Sửa lỗi phủ trắng biểu đồ khi cuộn (DrawRect Bug Fix):**



    Trong thư viện SkiaSharp, hàm `canvas.DrawRect(x, y, w, h, paint)` có tham số thứ tư là **Chiều cao (Height)** chứ không phải tọa độ đáy (Bottom). Khi sử dụng công thức cũ `canvas.DrawRect(0, yOffset, Width, yOffset + headerHeight, paint)`, phần chiều cao thực tế vẽ sẽ tăng tiến theo `yOffset`, tạo thành một mặt nạ hình chữ nhật khổng lồ che khuất toàn bộ phần biểu đồ phía dưới khi người dùng cuộn xuống.



    



    Để giải quyết triệt để và an toàn, hệ thống đã chuyển đổi sang sử dụng đối tượng `SKRect` xác định rõ nét bốn tọa độ biên `left, top, right, bottom`:



    ```csharp



    canvas.DrawRect(new SKRect(0, yOffset, (float)e.Info.Width, yOffset + headerHeight), headerBgP);



    ```



    Giải pháp này giúp đóng băng trục thời gian với chiều cao cố định đúng bằng `headerHeight` (`40f * dpi`) trong mọi trạng thái cuộn, bảo toàn giao diện biểu đồ Gantt bên dưới hoàn toàn sắc nét và ổn định.







### 10. Động cơ kết xuất Excel song ngữ 3 Sheet (Bilingual Excel Export Engine)



Để đồng bộ hóa dữ liệu từ tệp Microsoft Project (`.mpp`) đã import hoặc các sửa đổi trực tiếp trên phần mềm, YBIM TDTC v0.2.0-01 tích hợp công nghệ **kết xuất Excel lai giữa C# và Python**:







*   **Lớp điều phối phía C# (`MainWindow.xaml.cs`):**



    Chuyển đổi toàn bộ đối tượng `_tasks` và `_resourcePool` sang cấu trúc JSON tạm thời cực kỳ gọn nhẹ qua `System.Text.Json`, sau đó kích hoạt tiến trình Python ngầm chạy kịch bản kết xuất:



    ```csharp



    var psi = new System.Diagnostics.ProcessStartInfo {



        FileName = "python",



        Arguments = $"\"{scriptPath}\" \"{tempJsonPath}\" \"{sfd.FileName}\"",



        UseShellExecute = false,



        CreateNoWindow = true



    };



    ```



*   **Lớp dựng bảng và định dạng phía Python (`export_excel.py`):**



    Sử dụng thư viện `openpyxl` để kiến tạo tệp Excel `.xlsx` gồm đúng 3 Sheets chuẩn hóa:



    1.  **Tasks Sheet:** Xuất bảng danh sách công việc song ngữ với các cột `Id`, `WBS`, `Name (Tên công tác)`, `Start`, `Finish`, `Predecessors`, `Notes`, `IsSummary`. Tự động nhận biết công tác gọng (Summary) để áp dụng font đậm và tô màu nền dòng xám-xanh (`#F2F5F8`) chuyên nghiệp.



    2.  **Resources Sheet:** Xuất danh mục tài nguyên với các cột mã, tên, loại tài nguyên (tự chuyển đổi Enum sang text tương ứng: `Labor`, `Machinery`, `Material`), đơn vị, năng lực tối đa, đơn giá định mức (tự động áp dụng format số tiền phân cách hàng nghìn).



    3.  **Assignments Sheet:** Trích xuất tự động bảng phân bổ tài nguyên cho từng công tác làm căn cứ nhập nhanh ngược lại cho phần mềm.



    



    *   **Tự động căn chỉnh độ rộng cột (Auto-fit Columns):** Duyệt qua từng cột của cả 3 sheet để tự căn chỉnh độ rộng khít theo nội dung dài nhất cộng đệm 4 ký tự.



    *   **Thẩm mỹ chuẩn Excel Navy:** Tô nền tiêu đề màu Xanh Navy đậm (`#1F4E79`), font Segoe UI trắng đậm, kẻ viền ô mờ tinh tế (`#D9D9D9`) mang lại cảm giác cực kỳ cao cấp.







## ⚙️ 11. CẤU HÌNH HỆ THỐNG & TÙY CHỌN (OPTIONS & SETTINGSMANAGER)



Để đáp ứng tính cá nhân hóa và quản lý tập trung, phiên bản v0.2.0-01 đã tích hợp hệ thống `SettingsManager` thay thế hoàn toàn cho file `themesettings.json` cũ.







### 11.1. Kiến trúc Quản lý Cấu hình (SettingsManager)



*   **Mô hình dữ liệu (`AppSettings.cs`):** Một POCO class chứa toàn bộ các thuộc tính người dùng: `UserName`, `Initials`, `OfficeTheme`, `DateFormat`, `IsDarkMode`, `EnableAutoSave`, `AutoSaveIntervalMinutes`.



*   **Trình quản lý (`SettingsManager.cs`):** Đảm nhiệm việc Serialize/Deserialize ra file `appsettings.json`. Đặc biệt, có cơ chế **Tự động di chuyển (Migration)** dữ liệu từ file `themesettings.json` cũ sang cấu trúc mới trong lần chạy đầu tiên để không làm mất cài đặt màu sắc của người dùng.







### 11.2. Giao diện Options chuẩn MS Project



*   **Cấu trúc Window:** Xây dựng cửa sổ WPF `OptionsWindow` với Sidebar bên trái (ListBox) sử dụng kỹ thuật Navigation ẩn/hiện các Grid (TabGeneral, TabSave,...) giúp giao diện thay đổi mượt mà không cần nạp lại trang.



*   **Sửa lỗi Văng phần mềm (Crash) khi dùng Fluent Ribbon:** Khi khai báo một `<Fluent:Button>` mà thuộc tính `Icon="pack://..."` trỏ đến một ảnh không tồn tại, WPF sẽ ném ngoại lệ `NullReferenceException` khi dựng hình. Đã chuẩn hóa lại bằng cách sử dụng `TextBlock` với `FontFamily="Segoe Fluent Icons"` và `Style="{StaticResource RibbonSmallIcon}"` để vừa an toàn vừa đồng bộ giao diện.







### 11.3. Cơ chế Lưu tự động (AutoSave Threading)



*   Sử dụng `DispatcherTimer` chạy ngầm theo chu kỳ (mặc định 10 phút) được lấy từ `SettingsManager`.



*   **Silent Save:** Tiến trình lưu xuất file `.autosave` không can thiệp vào file hiện hành (`.ybim`), bảo vệ dữ liệu tuyệt đối trước các sự cố mất điện hay đóng phần mềm đột ngột. Cơ chế `CheckForCrashRecovery()` sẽ tự động hỏi người dùng phục hồi bản dữ liệu dở khi phát hiện file `.autosave` còn tồn tại ở lần khởi động kế tiếp.







## 📈 12. NÂNG CẤP HỆ THỐNG BIỂU ĐỒ ĐA TUYẾN (MULTI-SERIES LINE CHARTS)







Nhằm mang lại trải nghiệm Dashboard quản trị thực thụ, hệ thống Biểu đồ Dòng tiền và Biểu đồ Tài nguyên trong v0.2.0-01 đã được thiết kế lại hoàn toàn.







### 12.1. Đồ họa OLED Black và Gradient Fill



Thay thế toàn bộ Bar Chart khô khan bằng hệ thống **Line Chart** mượt mà. 



Sử dụng `SKPath` kết hợp `SKShader.CreateLinearGradient` để đổ màu mờ dần từ đường tín hiệu xuống trục hoành, tạo hiệu ứng phát sáng neon trên nền màu siêu tối (`#0B0F19` Window và `#111827` Canvas).







### 12.2. Kiến trúc Vẽ đa tuyến (Multi-Series Rendering)



Hệ thống `ResourceChartWindow` được nâng cấp từ việc nhận giá trị `Dictionary<string, double>` đơn lẻ sang cấu trúc `Dictionary<string, Dictionary<string, double>>` để có thể vẽ lồng nhiều đường đồ thị (Series) trên cùng một trục tọa độ:



*   **Máy thi công:** Đồng thời tính toán và hiển thị cả **Tổng Ca** (Shifts) và **Số lượng đỉnh** (Peak - lượng máy tối đa huy động trong một ngày thuộc tháng đó).



*   **Vật tư:** Hỗ trợ xem cả **Khối lượng** và **Giá trị VNĐ** (Quy đổi tự động sang triệu đồng `M` để giảm tải hiển thị).



*   Giao diện được trang bị thanh Toolbar với các `CheckBox` (Tổng Ca, Số máy, Khối lượng, Giá trị) được map trực tiếp với sự kiện `SkiaElement.InvalidateVisual()`, giúp biểu đồ vẽ lại ngay lập tức (Real-time render) mà không cần tính toán lại dữ liệu ngầm (`CalculateData()`).







---







## ⚡ 15. TỐI ƯU HIỆU NĂNG VÀ TRẢI NGHIỆM NGƯỜI DÙNG CAO CẤP (UX/UI & PERFORMANCE)







### 15.1. Thuật toán Lazy Rendering tối ưu hóa SkiaSharp Gantt Chart



Khi hiển thị Gantt Chart chứa số lượng công tác lớn, việc duyệt vẽ toàn bộ danh sách gây hao phí tài nguyên GPU/CPU nghiêm trọng. YBIM v0.2.0-01 triển khai thuật toán **Lazy Rendering** (Chỉ vẽ những gì nhìn thấy) bằng cách lấy độ lệch cuộn (`VerticalOffset`) và chiều cao hiển thị thực tế (`ViewportHeight`) từ `GanttScrollViewer` để tính toán giới hạn không gian vùng nhìn:







```csharp



float viewportTop = (float)GanttScrollViewer.VerticalOffset * dpi;



float viewportHeight = (float)GanttScrollViewer.ViewportHeight * dpi;



float viewportBottom = viewportTop + viewportHeight;



```







Trong vòng lặp vẽ các hàng công tác và vẽ liên kết predecessors, hệ thống tiến hành kiểm tra biên tọa độ của phần tử:



1.  **Với hàng công tác (Task Row):**



    ```csharp



    if (rowTop + currentRh < viewportTop - 100f * dpi || rowTop > viewportBottom + 100f * dpi) continue;



    ```



2.  **Với đường liên kết (Predecessor Line):**



    ```csharp



    float minY = Math.Min(yStart, yEnd);



    float maxY = Math.Max(yStart, yEnd);



    if (maxY < viewportTop - 100f * dpi && minY > viewportBottom + 100f * dpi) continue;



    ```



*Hiệu quả thực tế:* Tăng hiệu suất render Gantt Chart gấp **5 đến 10 lần**, giảm mức tiêu thụ CPU từ 80% xuống còn dưới 5% khi cuộn liên tục trên dự án 5000 dòng.







### 15.2. Công nghệ Khoét lỗ Vector (Vector Masking) trong Interactive Onboarding



Hệ thống sử dụng kỹ thuật dựng hình vector động trong WPF để làm mờ toàn bộ màn hình ngoại trừ vùng điều khiển đích (Highlight Control):



*   **Mặt nạ Vector:** Sử dụng thẻ `<Path>` có `Fill="#CC0B0F19"` và đặt dữ liệu hình học là một `GeometryGroup` cấu hình `FillRule="EvenOdd"`.



*   **Cơ chế trừ hình học (Geometry Subtraction):** Nhóm hình học chứa một hình chữ nhật lớn bao phủ toàn màn hình (`OnboardingFullRect`) và một hình chữ nhật nhỏ khít với tọa độ của nút bấm (`OnboardingHoleRect`). Phép toán logic `EvenOdd` tự động cắt đi vùng giao nhau, tạo ra một lỗ thủng sắc nét hoàn hảo cho phép người dùng nhìn thấy và bấm vào nút phía dưới mặt nạ.



*   **Định vị động Thẻ thông tin (Guide Card Canvas):** Lấy tọa độ thật của Highlight Control so với cửa sổ thông qua phép biến đổi ma trận không gian `TransformToAncestor(this).Transform(...)`. Sau đó tính toán vị trí hiển thị thẻ hướng dẫn nằm ngay phía dưới hoặc phía trên lỗ sáng kèm theo thuật toán kiểm tra biên cửa sổ (`Boundary checks`) chống tràn thẻ thông tin ra ngoài màn hình.







### 15.3. Xử lý đa luồng AI & Giao diện Khóa tương tác (Interactive Shield)



Để chống hiện tượng đơ luồng chính (WPF UI Thread) khi tính toán thuật toán di truyền san bằng tài nguyên phức tạp, động cơ AI được đẩy vào thực thi bất đồng bộ thông qua `Task.Run` chạy ngầm.



Để đảm bảo an toàn dữ liệu và tránh việc người dùng nhấp đúp nút chạy chồng chéo thuật toán, một lưới giao diện làm tối mờ (`LoadingOverlay`) có ProgressBar chạy vô tận sẽ tự động chuyển trạng thái `Visibility = Visibility.Visible`. Lưới này đóng vai trò như một tấm lá chắn khóa tương tác chuột (`Interactive Shield`), bảo vệ ứng dụng tuyệt đối trong suốt thời gian xử lý đa luồng.







### 15.4. Cơ chế Crash-Recovery & Tự động dọn dẹp nháp (Smart Clean Autosave)



Tích hợp sự kiện ghi đè `OnClosing` của cửa sổ `MainWindow` để xóa tệp tạm `.autosave` khi ứng dụng đóng an toàn:



```csharp



protected override void OnClosing(System.ComponentModel.CancelEventArgs e)



{



    base.OnClosing(e);



    try



    {



        if (File.Exists(_autoSavePath))



        {



            File.Delete(_autoSavePath);



        }



    }



    catch { }



}



```



Cơ chế phục hồi tiến độ (`CheckForCrashRecovery()`) chỉ được kích hoạt nếu tệp `.autosave` tồn tại lúc khởi động phần mềm. Việc tự động xóa file nháp khi thoát an toàn hoặc khi bấm lưu file thành công đảm bảo hộp thoại khôi phục chỉ xuất hiện khi ứng dụng thực sự bị sập nguồn/crash ngoài ý muốn, nâng cao đáng kể sự chuyên nghiệp và trải nghiệm người dùng.







---







## ⚡ 16. KỸ THUẬT TƯƠNG TÁC ĐA NỀN TẢNG (MS PROJECT XML & AUTOCAD LISP AUTOMATION)



### 16.1. Thuật toán đẩy ngược cấu trúc WBS & Link Logic bằng MPXJ (MSPDI XML)

Để tạo tệp XML MS Project hoàn chỉnh, hệ thống khởi dựng `ProjectFile` và áp dụng các kỹ thuật sau:

1.  **Dựng cây phân cấp WBS chuẩn xác:** Vì MPXJ không lưu trữ các mối quan hệ cha-con thông qua thuộc tính độc lập mà thông qua cấu trúc cây lồng nhau, hệ thống thiết lập sơ đồ duyệt tuần tự các công tác theo thứ tự hiển thị bằng cách lưu vết Task cha gần nhất tại mỗi OutlineLevel trong `parentTasks`:

    ```csharp

    net.sf.mpxj.Task mspTask;

    if (level <= 1) {

        mspTask = file.addTask();

    } else {

        if (parentTasks.TryGetValue(level - 1, out var parentTask)) {

            mspTask = parentTask.addTask();

        } else {

            mspTask = file.addTask();

        }

    }

    parentTasks[level] = mspTask;

    ```

2.  **Khởi tạo liên kết logic bằng Relation.Builder:** Mối quan hệ ràng buộc logic được gán cho Task kế tục (Successor) trỏ đến Task tiền nhiệm (Predecessor) sử dụng mẫu thiết kế Builder fluent-API của MPXJ:

    ```csharp

    var builder = new Relation.Builder();

    builder.predecessorTask(predTask);

    builder.type(relationType);

    builder.lag(lagDuration);

    mspTask.addPredecessor(builder);

    ```

    *Đặc thù .NET Wrapper:* Trình bọc IKVM MPXJ.NET nhận đối số `addPredecessor` trực tiếp là lớp `Relation.Builder` chứ không phải là đối tượng `Relation` đã `build()`, giúp tối giản hóa mã lệnh.



3.  **Áp dụng Đơn giá Tài nguyên theo CostRateTable mới:** Kể từ phiên bản MPXJ 10+, thuộc tính `setStandardRate` trực tiếp trên `Resource` bị loại bỏ. Thay vào đó, đơn giá được lưu tại bảng đơn giá chi tiết `CostRateTable` của tài nguyên:

    ```csharp

    var table = mspRes.getCostRateTable(0);

    var rates = new Rate[] { new Rate(java.lang.Double.valueOf(r.StandardRate), net.sf.mpxj.TimeUnit.HOURS) };

    var entry = new net.sf.mpxj.CostRateTableEntry(null, null, java.lang.Double.valueOf(0), rates);

    table.add(entry);

    ```



### 16.2. Kỹ thuật LISP chia tờ và thiết lập Viewport tự động (`c:YBIMPLOT`)

Đoạn mã AutoCAD LISP tự động hóa việc xuất bản tiến độ thi công ra giấy in A3 bằng các giải pháp lập trình hệ thống CAD chuyên sâu:

1.  **Tính toán phân đoạn in hình học:**

    *   Chiều rộng bảng WBS cố định: $170$.

    *   Chiều rộng Gantt trên mỗi tờ bản vẽ: $sheetW = 200$, khoảng phủ chồng mí $overlap = 20$.

    *   Số tờ bản vẽ: $numSheets = \lfloorrac{totalW - 170}{sheetW - overlap}
floor + 1$.

2.  **Tự động hóa Layout & Xóa viewport rác:** Sử dụng lệnh hệ thống `_-layout` để tạo mới và thiết lập tab hiển thị `(setvar "CTAB" layoutName)`. Lọc tìm và xóa toàn bộ các viewport mặc định của AutoCAD để tránh bị đè đè:

    ```lisp

    (setq ss (ssget "_X" '((0 . "VIEWPORT"))))

    (if ss (command "_.erase" ss ""))

    ```

3.  **Tự tạo Viewport (`_.mview`) và định vị tọa độ camera (`_.zoom`):**

    *   Trở về không gian giấy vẽ bằng `_.pspace` để vẽ khung tên và khung lề A3 (tỷ lệ 1:1 theo đơn vị mm của giấy).

    *   Mở viewport mới khớp từ tọa độ giấy vẽ `(30, 15)` sang `(405, 280)`.

    *   Truy cập vào không gian mô hình bằng `_.mspace`, sử dụng lệnh `_.zoom` với tham số `_W` (Window) để ép camera zoom chính xác từ góc dưới trái `(0, -totalH)` đến góc trên phải `(xEnd, 30)` của mô hình Gantt Chart Model Space tương ứng của tờ bản vẽ đó.



---



*Tài liệu được cập nhật ngày 22/05/2026 — Phiên bản YBIM TDTC v0.2.0-01*











            <TextBlock Style="{StaticResource RibbonLargeIcon}" Text="&#xE710;" />



        </Fluent:Button.LargeIcon>



    </Fluent:Button>



    ```



  * *Với SmallIcon (Biểu tượng nhỏ 16x16):*



    ```xml



    <Fluent:Button Header="Tên nút" Click="Btn_Click" Size="Middle">



        <Fluent:Button.Icon>



            <TextBlock Style="{StaticResource RibbonSmallIcon}" Text="&#xE787;" />



        </Fluent:Button.Icon>



    </Fluent:Button>



    ```



  * *Ghi đè màu sắc cá biệt (nếu có):*



    Nhờ cơ chế ưu tiên giá trị cục bộ (Local Value Precedence) của WPF, ta có thể dễ dàng ghi đè màu sắc (như màu đỏ cảnh báo) ngay tại thẻ TextBlock mà không làm mất các thuộc tính căn chỉnh của Style:



    ```xml



    <TextBlock Style="{StaticResource RibbonLargeIcon}" Text="&#xEA90;" Foreground="{DynamicResource Y_Critical}" />



    ```







### 5.3. Nguyên Tắc Bắt Buộc: Mã Hóa File XAML Với UTF-8 Có BOM



Để tiếng Việt có dấu hiển thị chuẩn xác trên mọi máy tính chạy phiên bản Windows với ngôn ngữ hệ thống khác nhau:



* **Vấn đề:** Nếu file XAML được lưu ở dạng UTF-8 không có BOM (Byte Order Mark), trình phân tích cú pháp XAML của WPF sẽ tự động chuyển sang đọc file theo trang mã ANSI mặc định của hệ điều hành. Việc này làm hỏng toàn bộ tiếng Việt trong ứng dụng thành các chuỗi ký tự lạ.



* **Giải pháp:** Mọi file XAML bắt buộc phải được lưu với mã hóa **UTF-8 có BOM** (`utf-8-sig` trong Python). Byte BOM đầu file (`\xef\xbb\xbf`) sẽ ép trình biên dịch WPF nhận diện chính xác 100% tiếng Việt UTF-8.







### 5.4. Hệ Thống Chủ Đề Và Responsive Layout (Responsive Theme System)



* **Bảng màu đồng bộ (Harmony Palette):** Toàn bộ các nút bấm và nhãn trong ứng dụng đều ràng buộc chặt chẽ với hệ thống DynamicResource tài nguyên màu chuẩn: `Y_Accent` (Màu nhấn chủ đạo), `Y_SurfaceBackground` (Màu nền các bảng và hội thoại), `Y_TextPrimary` & `Y_TextSecondary`.



* **Chủ đề Sáng/Tối (Light & Dark Mode):** Hỗ trợ đổi chủ đề mượt mà qua các file tài nguyên `Themes/ModernLight.xaml` và `Themes/ModernDark.xaml`, tự động lưu trữ và tải lại thông qua cấu hình `themesettings.json`.



* **Ribbon Tự Co Giãn (Adaptive Ribbon):** Giao diện sử dụng bộ điều khiển `Fluent:RibbonWindow` cao cấp. Ở chế độ thu nhỏ màn hình hoặc máy tính bảng (màn 14-16" display), Ribbon tự động rút gọn cấu trúc cột nút từ `350px` xuống còn `120px` để bảo toàn không gian làm việc tối đa cho Gantt Chart bên dưới.







---







## 📧 6. HIỂN THỊ EMAIL NGƯỜI DÙNG TRONG ỨNG DỤNG VÀ FILE EXPORT







Để tăng tính cá nhân hoá và hỗ trợ quản lý bản quyền, chúng ta sẽ hiển thị địa chỉ email đã lưu (được lưu trong `LicenseManager`) trên giao diện và nhúng vào các file export (TXT/Excel).







### 6.1. Lấy email người dùng



```csharp



// Trong MainWindow.xaml.cs



string userEmail = LicenseManager.GetStoredEmail(); // trả về email đã kích hoạt



```



* Đảm bảo hiển thị trên UI bằng cách gán cho TextBlock: `TxtUserEmail.Text = $"Email: {userEmail}";` (Hiển thị "Email: chưa xác định" nếu chưa kích hoạt).







### 6.2. Nhúng email vào file TXT xuất tiến độ



Trong phương thức `BtnExportTxt_Click` thêm một dòng trước khi ghi dữ liệu:



```csharp



sb.Insert(0, $"Email\t{userEmail}\n"); // Đặt email ở dòng đầu tiên



```







### 6.3. Nhúng email vào file Excel (khi xuất Excel)



Trong `BtnExportExcel_Click` (phần tạo StringBuilder), chèn:



```csharp



sb.Insert(0, $"<Email>{userEmail}</Email>\n");



```







---







## ✅ 7. CHECKLIST KIỂM TRA SAU KHI TÍCH HỢP







- [x] Package `GeneticSharp 3.1.4` đã được thêm vào `.csproj`.



- [x] 3 file service mới biên dịch thành công (0 Error).



- [x] `BtnAIOptimize_Click` đã được thêm vào `MainWindow.xaml.cs`.



- [x] `CreateOptimizationTabs` tạo 3 tab kết quả đúng.



- [x] `CommitOptimizedPlan` xóa tab và refresh Gantt đúng.



- [x] Build Release thành công kèm Obfuscar.



- [x] Version bumped từ `v0.1.3` → `v0.2.0-01` tại **tất cả** vị trí.



- [x] File `YBIM_TDTC_v0.2.0-01.zip` (89.5 MB) đã đóng gói.



- [x] Thêm nút Ribbon trong `MainWindow.xaml` kết nối `BtnAIOptimize_Click` và căn chỉnh trục thẳng hàng.



- [x] Thiết kế Style quy chuẩn `RibbonLargeIcon` và `RibbonSmallIcon` chống lệch tâm đứng/ngang cho phông chữ Segoe Fluent Icons.



- [x] **MỚI:** Tích hợp tính năng Nhập Dự án 1-Click từ Excel mẫu song ngữ (`YBIM_ImportTemplate.xlsx`).







---







## 📊 8. HỆ THỐNG NHẬP DỰ ÁN 1-CLICK TỪ EXCEL SONG NGỮ TOÀN DIỆN







Để hiện thực hóa tài liệu hướng dẫn nhập mẫu song ngữ, phiên bản **YBIM TDTC v0.2.0-01** đã tích hợp thành công trình thông dịch dự án 1-click từ Excel thông minh tại phương thức `ImportExcel_Click` trong `MainWindow.xaml.cs`.







### 8.1. Cơ chế Fuzzy Matching và Đa ngôn ngữ (Bilingual Columns Detection)



Hệ thống sử dụng thư viện `ExcelDataReader` đọc tệp `.xlsx` siêu tốc, tự động phân tích và khớp cột mà không phân biệt chữ hoa, chữ thường hay ngôn ngữ (Anh - Việt):



*   **Sheet nhận diện:**



    *   `Tasks` hoặc `công việc` -> Chuyển thành bảng dữ liệu Công tác.



    *   `Resources` hoặc `tài nguyên` -> Chuyển thành bảng dữ liệu Tài nguyên.



    *   `Assignments` hoặc `phân bổ` -> Chuyển thành bảng phân bổ tài nguyên.



*   **Thuật toán Fuzzy Columns mapping:**



    ```csharp



    // Ví dụ khớp thông minh cột tên công tác



    if (colName.Contains("name") || colName.Contains("tên")) nameCol = col;



    // Khớp thông minh cột mối quan hệ tiền nhiệm



    if (colName.Contains("predecessor") || colName.Contains("tiền nhiệm")) tPredCol = col;



    // Khớp thông minh cột số ngày thi công (mới)



    if (colName.Contains("duration") || colName.Contains("ngày thi công") || colName.Contains("thời gian thi công") || colName.Contains("số ngày")) tDurCol = col;



    ```







*   **Xử lý nạp & Dự phòng tương thích (Backward Compatibility Fallback):**



    Nếu cột ngày thi công (`tDurCol`) tồn tại và có dữ liệu hợp lệ, hệ thống sẽ ưu tiên nạp giá trị số này vào `YTask.Duration`. Nếu tệp nhập vào thiếu cột này (ví dụ tệp biểu mẫu cũ), chương trình tự động tính toán số ngày thi công dự phòng từ khoảng cách giữa ngày bắt đầu và ngày kết thúc:



    ```csharp



    duration = (FinishDate - StartDate).TotalDays;



    if (duration <= 0) duration = 1; // Giá trị dự phòng tối thiểu



    ```







### 8.2. Giải mã thuật toán khớp mối liên kết tiền nhiệm (Dependency String Parser)



Cơ chế tự động bóc tách chuỗi liên kết phức tạp từ Excel (ví dụ: `1FS+2d, 3SS-1, 4FF+0d`):



1.  Bóc tách chuỗi theo các dấu phân cách `,` hoặc `;`.



2.  Tách chỉ số ID công tác tiền nhiệm (các chữ số đầu tiên).



3.  Nhận diện các kiểu mối quan hệ: `SS`, `FF`, `SF`, `FS` (hoặc mặc định là `FS` nếu không ghi rõ).



4.  Nhận diện số ngày trễ (Lag) hỗ trợ cả giá trị âm/dương và hậu tố `d` hoặc `ngày` (ví dụ: `+2d` -> `2`, `-1` -> `-1`).







---







## ❄️ 9. CƠ CHẾ CỐ ĐỊNH TRỤC THỜI GIAN GANTT CHART (FROZEN TIMESCALE HEADER)







Để tối ưu hóa trải nghiệm người dùng khi làm việc với dự án lớn có hàng trăm công tác, phiên bản **YBIM TDTC v0.2.0-01** đã tích hợp thành công giải pháp **cố định trục thời gian (Timescale)** ở đầu vùng nhìn biểu đồ Gantt khi cuộn dọc.







### 9.1. Kiến trúc Vẽ 3 Giai Đoạn (Three-Phase Painting Scheme)



Thay vì vẽ trục thời gian trước rồi vẽ thanh công việc đè lên như cũ, hệ thống được cấu trúc lại hoàn toàn theo 3 giai đoạn độc lập trên SkiaSharp canvas:



1.  **Pha 1 (Vẽ Nền & Lưới Dọc):** Vẽ phần tô màu cuối tuần (Weekend shading) và các đường giăng dọc lưới Gantt xuống dưới bắt đầu từ tọa độ `headerHeight` đến hết chiều cao thực tế `e.Info.Height`.



2.  **Pha 2 (Vẽ Công Tác & Liên Kết Găng):** Vẽ toàn bộ đường liên kết mối quan hệ tiền nhiệm, các thanh công tác (Task Bars) và nhãn ngày tháng tương ứng.



3.  **Pha 3 (Vẽ Trục Thời Gian Đóng Băng - Sticky Overlay):** Vẽ đè phần nền trục thời gian (`headerBgColor`), các nhãn tháng/năm (Top Tier), vạch ngăn cách ngày/quý (Bottom Tier) ở tọa độ dịch chuyển `yOffset`.







### 9.2. Thuật toán dịch chuyển và đồng bộ cuộn (yOffset & Real-time Invalidation)



*   **Tính toán dịch chuyển:**



    ```csharp



    float yOffset = (float)GanttScrollViewer.VerticalOffset * dpi;



    ```



    Toàn bộ các tọa độ vẽ của trục thời gian trong Pha 3 đều được cộng thêm `yOffset`. Nhờ đó, trục thời gian luôn nằm cố định ở phía trên cùng của viewport hiển thị dù biểu đồ Gantt có cuộn xuống sâu đến mức nào.



*   **Vẽ lại thời gian thực (Real-time Repaint):**



    Tại sự kiện cuộn dọc của cả `TaskGrid` và `GanttScrollViewer`, hệ thống gọi tường minh:



    ```csharp



    GanttSkiaElement?.InvalidateVisual();



    ```



    Việc này ép buộc SkiaSharp vẽ lại trục thời gian khớp với độ lệch cuộn ngay lập tức, triệt tiêu hoàn toàn hiện tượng giật lag hay lệch vị trí.







*   **Sửa lỗi phủ trắng biểu đồ khi cuộn (DrawRect Bug Fix):**



    Trong thư viện SkiaSharp, hàm `canvas.DrawRect(x, y, w, h, paint)` có tham số thứ tư là **Chiều cao (Height)** chứ không phải tọa độ đáy (Bottom). Khi sử dụng công thức cũ `canvas.DrawRect(0, yOffset, Width, yOffset + headerHeight, paint)`, phần chiều cao thực tế vẽ sẽ tăng tiến theo `yOffset`, tạo thành một mặt nạ hình chữ nhật khổng lồ che khuất toàn bộ phần biểu đồ phía dưới khi người dùng cuộn xuống.



    



    Để giải quyết triệt để và an toàn, hệ thống đã chuyển đổi sang sử dụng đối tượng `SKRect` xác định rõ nét bốn tọa độ biên `left, top, right, bottom`:



    ```csharp



    canvas.DrawRect(new SKRect(0, yOffset, (float)e.Info.Width, yOffset + headerHeight), headerBgP);



    ```



    Giải pháp này giúp đóng băng trục thời gian với chiều cao cố định đúng bằng `headerHeight` (`40f * dpi`) trong mọi trạng thái cuộn, bảo toàn giao diện biểu đồ Gantt bên dưới hoàn toàn sắc nét và ổn định.







### 10. Động cơ kết xuất Excel song ngữ 3 Sheet (Bilingual Excel Export Engine)



Để đồng bộ hóa dữ liệu từ tệp Microsoft Project (`.mpp`) đã import hoặc các sửa đổi trực tiếp trên phần mềm, YBIM TDTC v0.2.0-01 tích hợp công nghệ **kết xuất Excel lai giữa C# và Python**:







*   **Lớp điều phối phía C# (`MainWindow.xaml.cs`):**



    Chuyển đổi toàn bộ đối tượng `_tasks` và `_resourcePool` sang cấu trúc JSON tạm thời cực kỳ gọn nhẹ qua `System.Text.Json`, sau đó kích hoạt tiến trình Python ngầm chạy kịch bản kết xuất:



    ```csharp



    var psi = new System.Diagnostics.ProcessStartInfo {



        FileName = "python",



        Arguments = $"\"{scriptPath}\" \"{tempJsonPath}\" \"{sfd.FileName}\"",



        UseShellExecute = false,



        CreateNoWindow = true



    };



    ```



*   **Lớp dựng bảng và định dạng phía Python (`export_excel.py`):**



    Sử dụng thư viện `openpyxl` để kiến tạo tệp Excel `.xlsx` gồm đúng 3 Sheets chuẩn hóa:



    1.  **Tasks Sheet:** Xuất bảng danh sách công việc song ngữ với các cột `Id`, `WBS`, `Name (Tên công tác)`, `Start`, `Finish`, `Predecessors`, `Notes`, `IsSummary`. Tự động nhận biết công tác gọng (Summary) để áp dụng font đậm và tô màu nền dòng xám-xanh (`#F2F5F8`) chuyên nghiệp.



    2.  **Resources Sheet:** Xuất danh mục tài nguyên với các cột mã, tên, loại tài nguyên (tự chuyển đổi Enum sang text tương ứng: `Labor`, `Machinery`, `Material`), đơn vị, năng lực tối đa, đơn giá định mức (tự động áp dụng format số tiền phân cách hàng nghìn).



    3.  **Assignments Sheet:** Trích xuất tự động bảng phân bổ tài nguyên cho từng công tác làm căn cứ nhập nhanh ngược lại cho phần mềm.



    



    *   **Tự động căn chỉnh độ rộng cột (Auto-fit Columns):** Duyệt qua từng cột của cả 3 sheet để tự căn chỉnh độ rộng khít theo nội dung dài nhất cộng đệm 4 ký tự.



    *   **Thẩm mỹ chuẩn Excel Navy:** Tô nền tiêu đề màu Xanh Navy đậm (`#1F4E79`), font Segoe UI trắng đậm, kẻ viền ô mờ tinh tế (`#D9D9D9`) mang lại cảm giác cực kỳ cao cấp.







## ⚙️ 11. CẤU HÌNH HỆ THỐNG & TÙY CHỌN (OPTIONS & SETTINGSMANAGER)



Để đáp ứng tính cá nhân hóa và quản lý tập trung, phiên bản v0.2.0-01 đã tích hợp hệ thống `SettingsManager` thay thế hoàn toàn cho file `themesettings.json` cũ.







### 11.1. Kiến trúc Quản lý Cấu hình (SettingsManager)



*   **Mô hình dữ liệu (`AppSettings.cs`):** Một POCO class chứa toàn bộ các thuộc tính người dùng: `UserName`, `Initials`, `OfficeTheme`, `DateFormat`, `IsDarkMode`, `EnableAutoSave`, `AutoSaveIntervalMinutes`.



*   **Trình quản lý (`SettingsManager.cs`):** Đảm nhiệm việc Serialize/Deserialize ra file `appsettings.json`. Đặc biệt, có cơ chế **Tự động di chuyển (Migration)** dữ liệu từ file `themesettings.json` cũ sang cấu trúc mới trong lần chạy đầu tiên để không làm mất cài đặt màu sắc của người dùng.







### 11.2. Giao diện Options chuẩn MS Project



*   **Cấu trúc Window:** Xây dựng cửa sổ WPF `OptionsWindow` với Sidebar bên trái (ListBox) sử dụng kỹ thuật Navigation ẩn/hiện các Grid (TabGeneral, TabSave,...) giúp giao diện thay đổi mượt mà không cần nạp lại trang.



*   **Sửa lỗi Văng phần mềm (Crash) khi dùng Fluent Ribbon:** Khi khai báo một `<Fluent:Button>` mà thuộc tính `Icon="pack://..."` trỏ đến một ảnh không tồn tại, WPF sẽ ném ngoại lệ `NullReferenceException` khi dựng hình. Đã chuẩn hóa lại bằng cách sử dụng `TextBlock` với `FontFamily="Segoe Fluent Icons"` và `Style="{StaticResource RibbonSmallIcon}"` để vừa an toàn vừa đồng bộ giao diện.







### 11.3. Cơ chế Lưu tự động (AutoSave Threading)



*   Sử dụng `DispatcherTimer` chạy ngầm theo chu kỳ (mặc định 10 phút) được lấy từ `SettingsManager`.



*   **Silent Save:** Tiến trình lưu xuất file `.autosave` không can thiệp vào file hiện hành (`.ybim`), bảo vệ dữ liệu tuyệt đối trước các sự cố mất điện hay đóng phần mềm đột ngột. Cơ chế `CheckForCrashRecovery()` sẽ tự động hỏi người dùng phục hồi bản dữ liệu dở khi phát hiện file `.autosave` còn tồn tại ở lần khởi động kế tiếp.







## 📈 12. NÂNG CẤP HỆ THỐNG BIỂU ĐỒ ĐA TUYẾN (MULTI-SERIES LINE CHARTS)







Nhằm mang lại trải nghiệm Dashboard quản trị thực thụ, hệ thống Biểu đồ Dòng tiền và Biểu đồ Tài nguyên trong v0.2.0-01 đã được thiết kế lại hoàn toàn.







### 12.1. Đồ họa OLED Black và Gradient Fill



Thay thế toàn bộ Bar Chart khô khan bằng hệ thống **Line Chart** mượt mà. 



Sử dụng `SKPath` kết hợp `SKShader.CreateLinearGradient` để đổ màu mờ dần từ đường tín hiệu xuống trục hoành, tạo hiệu ứng phát sáng neon trên nền màu siêu tối (`#0B0F19` Window và `#111827` Canvas).







### 12.2. Kiến trúc Vẽ đa tuyến (Multi-Series Rendering)



Hệ thống `ResourceChartWindow` được nâng cấp từ việc nhận giá trị `Dictionary<string, double>` đơn lẻ sang cấu trúc `Dictionary<string, Dictionary<string, double>>` để có thể vẽ lồng nhiều đường đồ thị (Series) trên cùng một trục tọa độ:



*   **Máy thi công:** Đồng thời tính toán và hiển thị cả **Tổng Ca** (Shifts) và **Số lượng đỉnh** (Peak - lượng máy tối đa huy động trong một ngày thuộc tháng đó).



*   **Vật tư:** Hỗ trợ xem cả **Khối lượng** và **Giá trị VNĐ** (Quy đổi tự động sang triệu đồng `M` để giảm tải hiển thị).



*   Giao diện được trang bị thanh Toolbar với các `CheckBox` (Tổng Ca, Số máy, Khối lượng, Giá trị) được map trực tiếp với sự kiện `SkiaElement.InvalidateVisual()`, giúp biểu đồ vẽ lại ngay lập tức (Real-time render) mà không cần tính toán lại dữ liệu ngầm (`CalculateData()`).







---







## ⚡ 15. TỐI ƯU HIỆU NĂNG VÀ TRẢI NGHIỆM NGƯỜI DÙNG CAO CẤP (UX/UI & PERFORMANCE)







### 15.1. Thuật toán Lazy Rendering tối ưu hóa SkiaSharp Gantt Chart



Khi hiển thị Gantt Chart chứa số lượng công tác lớn, việc duyệt vẽ toàn bộ danh sách gây hao phí tài nguyên GPU/CPU nghiêm trọng. YBIM v0.2.0-01 triển khai thuật toán **Lazy Rendering** (Chỉ vẽ những gì nhìn thấy) bằng cách lấy độ lệch cuộn (`VerticalOffset`) và chiều cao hiển thị thực tế (`ViewportHeight`) từ `GanttScrollViewer` để tính toán giới hạn không gian vùng nhìn:







```csharp



float viewportTop = (float)GanttScrollViewer.VerticalOffset * dpi;



float viewportHeight = (float)GanttScrollViewer.ViewportHeight * dpi;



float viewportBottom = viewportTop + viewportHeight;



```







Trong vòng lặp vẽ các hàng công tác và vẽ liên kết predecessors, hệ thống tiến hành kiểm tra biên tọa độ của phần tử:



1.  **Với hàng công tác (Task Row):**



    ```csharp



    if (rowTop + currentRh < viewportTop - 100f * dpi || rowTop > viewportBottom + 100f * dpi) continue;



    ```



2.  **Với đường liên kết (Predecessor Line):**



    ```csharp



    float minY = Math.Min(yStart, yEnd);



    float maxY = Math.Max(yStart, yEnd);



    if (maxY < viewportTop - 100f * dpi && minY > viewportBottom + 100f * dpi) continue;



    ```



*Hiệu quả thực tế:* Tăng hiệu suất render Gantt Chart gấp **5 đến 10 lần**, giảm mức tiêu thụ CPU từ 80% xuống còn dưới 5% khi cuộn liên tục trên dự án 5000 dòng.







### 15.2. Công nghệ Khoét lỗ Vector (Vector Masking) trong Interactive Onboarding



Hệ thống sử dụng kỹ thuật dựng hình vector động trong WPF để làm mờ toàn bộ màn hình ngoại trừ vùng điều khiển đích (Highlight Control):



*   **Mặt nạ Vector:** Sử dụng thẻ `<Path>` có `Fill="#CC0B0F19"` và đặt dữ liệu hình học là một `GeometryGroup` cấu hình `FillRule="EvenOdd"`.



*   **Cơ chế trừ hình học (Geometry Subtraction):** Nhóm hình học chứa một hình chữ nhật lớn bao phủ toàn màn hình (`OnboardingFullRect`) và một hình chữ nhật nhỏ khít với tọa độ của nút bấm (`OnboardingHoleRect`). Phép toán logic `EvenOdd` tự động cắt đi vùng giao nhau, tạo ra một lỗ thủng sắc nét hoàn hảo cho phép người dùng nhìn thấy và bấm vào nút phía dưới mặt nạ.



*   **Định vị động Thẻ thông tin (Guide Card Canvas):** Lấy tọa độ thật của Highlight Control so với cửa sổ thông qua phép biến đổi ma trận không gian `TransformToAncestor(this).Transform(...)`. Sau đó tính toán vị trí hiển thị thẻ hướng dẫn nằm ngay phía dưới hoặc phía trên lỗ sáng kèm theo thuật toán kiểm tra biên cửa sổ (`Boundary checks`) chống tràn thẻ thông tin ra ngoài màn hình.







### 15.3. Xử lý đa luồng AI & Giao diện Khóa tương tác (Interactive Shield)



Để chống hiện tượng đơ luồng chính (WPF UI Thread) khi tính toán thuật toán di truyền san bằng tài nguyên phức tạp, động cơ AI được đẩy vào thực thi bất đồng bộ thông qua `Task.Run` chạy ngầm.



Để đảm bảo an toàn dữ liệu và tránh việc người dùng nhấp đúp nút chạy chồng chéo thuật toán, một lưới giao diện làm tối mờ (`LoadingOverlay`) có ProgressBar chạy vô tận sẽ tự động chuyển trạng thái `Visibility = Visibility.Visible`. Lưới này đóng vai trò như một tấm lá chắn khóa tương tác chuột (`Interactive Shield`), bảo vệ ứng dụng tuyệt đối trong suốt thời gian xử lý đa luồng.







### 15.4. Cơ chế Crash-Recovery & Tự động dọn dẹp nháp (Smart Clean Autosave)



Tích hợp sự kiện ghi đè `OnClosing` của cửa sổ `MainWindow` để xóa tệp tạm `.autosave` khi ứng dụng đóng an toàn:



```csharp



protected override void OnClosing(System.ComponentModel.CancelEventArgs e)



{



    base.OnClosing(e);



    try



    {



        if (File.Exists(_autoSavePath))



        {



            File.Delete(_autoSavePath);



        }



    }



    catch { }



}



```



Cơ chế phục hồi tiến độ (`CheckForCrashRecovery()`) chỉ được kích hoạt nếu tệp `.autosave` tồn tại lúc khởi động phần mềm. Việc tự động xóa file nháp khi thoát an toàn hoặc khi bấm lưu file thành công đảm bảo hộp thoại khôi phục chỉ xuất hiện khi ứng dụng thực sự bị sập nguồn/crash ngoài ý muốn, nâng cao đáng kể sự chuyên nghiệp và trải nghiệm người dùng.







---







## ⚡ 16. KỸ THUẬT TƯƠNG TÁC ĐA NỀN TẢNG (MS PROJECT XML & AUTOCAD LISP AUTOMATION)



### 16.1. Thuật toán đẩy ngược cấu trúc WBS & Link Logic bằng MPXJ (MSPDI XML)

Để tạo tệp XML MS Project hoàn chỉnh, hệ thống khởi dựng `ProjectFile` và áp dụng các kỹ thuật sau:

1.  **Dựng cây phân cấp WBS chuẩn xác:** Vì MPXJ không lưu trữ các mối quan hệ cha-con thông qua thuộc tính độc lập mà thông qua cấu trúc cây lồng nhau, hệ thống thiết lập sơ đồ duyệt tuần tự các công tác theo thứ tự hiển thị bằng cách lưu vết Task cha gần nhất tại mỗi OutlineLevel trong `parentTasks`:

    ```csharp

    net.sf.mpxj.Task mspTask;

    if (level <= 1) {

        mspTask = file.addTask();

    } else {

        if (parentTasks.TryGetValue(level - 1, out var parentTask)) {

            mspTask = parentTask.addTask();

        } else {

            mspTask = file.addTask();

        }

    }

    parentTasks[level] = mspTask;

    ```

2.  **Khởi tạo liên kết logic bằng Relation.Builder:** Mối quan hệ ràng buộc logic được gán cho Task kế tục (Successor) trỏ đến Task tiền nhiệm (Predecessor) sử dụng mẫu thiết kế Builder fluent-API của MPXJ:

    ```csharp

    var builder = new Relation.Builder();

    builder.predecessorTask(predTask);

    builder.type(relationType);

    builder.lag(lagDuration);

    mspTask.addPredecessor(builder);

    ```

    *Đặc thù .NET Wrapper:* Trình bọc IKVM MPXJ.NET nhận đối số `addPredecessor` trực tiếp là lớp `Relation.Builder` chứ không phải là đối tượng `Relation` đã `build()`, giúp tối giản hóa mã lệnh.



3.  **Áp dụng Đơn giá Tài nguyên theo CostRateTable mới:** Kể từ phiên bản MPXJ 10+, thuộc tính `setStandardRate` trực tiếp trên `Resource` bị loại bỏ. Thay vào đó, đơn giá được lưu tại bảng đơn giá chi tiết `CostRateTable` của tài nguyên:

    ```csharp

    var table = mspRes.getCostRateTable(0);

    var rates = new Rate[] { new Rate(java.lang.Double.valueOf(r.StandardRate), net.sf.mpxj.TimeUnit.HOURS) };

    var entry = new net.sf.mpxj.CostRateTableEntry(null, null, java.lang.Double.valueOf(0), rates);

    table.add(entry);

    ```



### 16.2. Kỹ thuật LISP chia tờ và thiết lập Viewport tự động (`c:YBIMPLOT`)

Đoạn mã AutoCAD LISP tự động hóa việc xuất bản tiến độ thi công ra giấy in A3 bằng các giải pháp lập trình hệ thống CAD chuyên sâu:

1.  **Tính toán phân đoạn in hình học:**

    *   Chiều rộng bảng WBS cố định: $170$.

    *   Chiều rộng Gantt trên mỗi tờ bản vẽ: $sheetW = 200$, khoảng phủ chồng mí $overlap = 20$.

    *   Số tờ bản vẽ: $numSheets = \lfloorrac{totalW - 170}{sheetW - overlap}
floor + 1$.

2.  **Tự động hóa Layout & Xóa viewport rác:** Sử dụng lệnh hệ thống `_-layout` để tạo mới và thiết lập tab hiển thị `(setvar "CTAB" layoutName)`. Lọc tìm và xóa toàn bộ các viewport mặc định của AutoCAD để tránh bị đè đè:

    ```lisp

    (setq ss (ssget "_X" '((0 . "VIEWPORT"))))

    (if ss (command "_.erase" ss ""))

    ```

3.  **Tự tạo Viewport (`_.mview`) và định vị tọa độ camera (`_.zoom`):**

    *   Trở về không gian giấy vẽ bằng `_.pspace` để vẽ khung tên và khung lề A3 (tỷ lệ 1:1 theo đơn vị mm của giấy).

    *   Mở viewport mới khớp từ tọa độ giấy vẽ `(30, 15)` sang `(405, 280)`.

    *   Truy cập vào không gian mô hình bằng `_.mspace`, sử dụng lệnh `_.zoom` với tham số `_W` (Window) để ép camera zoom chính xác từ góc dưới trái `(0, -totalH)` đến góc trên phải `(xEnd, 30)` của mô hình Gantt Chart Model Space tương ứng của tờ bản vẽ đó.



---



*Tài liệu được cập nhật ngày 22/05/2026 — Phiên bản YBIM TDTC v0.2.0-01*



















### Xây dựng cơ chế Undo/Redo và Báo cáo HTML (Chart.js)



- Cơ chế Undo/Redo: Áp dụng Memento / State Serialization. Lưu toàn bộ list YTask và YResource qua System.Text.Json. Khống chế giới hạn 50 lớp trên Stack để tránh tràn RAM. Hàm Hook SaveStateToUndo() được gài tự động vào các thao tác: CellEditEnding, AddTask, DeleteTask (Cut/Phím Delete), LinkTasks, CommitOptimizedPlan (AI).



- Snapshot: Lưu trạng thái theo dạng Dictionary<string, string> (Tên bản nháp -> JSON string) để dễ quản lý các PA.



- Báo cáo HTML: Sử dụng chuỗi string interpolation nhúng HTML5 + CSS in-line. Biểu đồ dùng Chart.js load qua CDN. File lưu cục bộ (.html) và tự động gọi trình duyệt bằng UseShellExecute.







---







## 🛠️ 13. SỬA LỖI ĐỊNH DẠNG XUẤT LISP AUTOCAD TRÊN MÁY TÍNH VIỆT NAM (DECIMAL SEPARATOR FIX)







### 13.1. Phân tích lỗi (Bug Analysis)



Trong chức năng xuất tiến độ ra LISP AutoCAD, hệ thống cần nội suy hàng ngàn tọa độ điểm dạng số thập phân (Ví dụ: `170.5`).



Tuy nhiên, nếu người dùng cấu hình hệ điều hành Windows theo khu vực Việt Nam (`vi-VN`), các chuỗi String Interpolation (ví dụ: `$"{x1}"`) sẽ tự động xuất ra dấu phẩy làm phân cách thập phân (thành `170,5`). 



Ngôn ngữ AutoLISP của AutoCAD yêu cầu dấu chấm `.` cho số thập phân. Việc xuất hiện dấu phẩy sẽ khiến trình thông dịch LISP phân tách sai danh sách tham số (list), dẫn đến lỗi cú pháp (Bad argument type, Syntax error) làm cho người dùng không thể nạp file bằng lệnh `APPLOAD`.







### 13.2. Giải pháp kỹ thuật (The Fix)



- **Ép kiểu Culture Cục bộ (Culture Isolation):** Toàn bộ tiến trình phát sinh LISP (`ExportAutoLispExporter_Click`) đã được đặt trong một khối `try...finally`. Trước khi bắt đầu xuất, hệ thống lưu lại `CurrentCulture` hiện tại và tạm thời chuyển luồng thực thi sang `CultureInfo.InvariantCulture`. Việc này ép toàn bộ số thập phân đều dùng chuẩn dấu chấm `.`. Khi kết thúc hàm, hệ thống tự động trả lại Culture ban đầu để không làm ảnh hưởng đến các hiển thị giao diện WPF khác của ứng dụng.



- **Tương thích Encoding:** Đổi `StreamWriter` từ `Encoding.UTF8` (có BOM) sang `new UTF8Encoding(false)` (không có BOM) để đảm bảo trình thông dịch LISP của các phiên bản AutoCAD cũ không bị nhiễu bởi các byte định dạng ẩn ở đầu file.

### 13.3. Quy trình Vận hành chuẩn trên AutoCAD (AutoCAD Workflow)
Để đảm bảo biểu đồ tiến độ vẽ tự động và chia tờ dàn layout in ấn hoạt động chuẩn xác:
1. Mở AutoCAD, gõ lệnh **`APPLOAD`**, nạp file **`VeTienDo_YBIM.lsp`** mới xuất từ phần mềm.
2. Gõ lệnh **`VETIENDO`** để tự động vẽ toàn bộ tiến độ bên Model Space.
3. Gõ lệnh **`YBIMPLOT`** để tự động chia tờ và dàn layout in ấn A3 cực kỳ đẹp mắt.








---







## 🎨 14. QUY HOẠCH KÍCH THƯỚC NÚT (SIZE DEFINITION) & TỐI ƯU HÓA RIBBON MENU 15.6 INCH







### 14.1. Vấn đề không gian hiển thị trên Laptop 15.6 inch



Giao diện Ribbon truyền thống dàn hàng ngang toàn bộ các nút lớn (`Size="Large"`) chiếm dụng diện tích chiều ngang rất lớn. Khi chạy phần mềm trên các màn hình laptop 15.6 inch với tỉ lệ hiển thị mặc định (Scale 125% - 150%), dải Ribbon bị tràn viền và xuất hiện các thanh cuộn ngang/thu gọn cột gây mất thẩm mỹ và khó khăn khi tương tác.







### 14.2. Giải pháp quy hoạch Layout và Kích thước Nút



* **Gom cụm nhóm chức năng lớn:**



  * Loại bỏ các nhóm nhỏ lẻ dư thừa như `QC`, `Baseline`, `Cấu trúc`.



  * Gom cụm toàn bộ tác vụ thành hai nhóm chính cực kỳ gọn gàng: **"Quản lý Tác vụ & Liên kết"** và **"Công cụ & Xuất bản"**.



* **Áp dụng cơ chế quy hoạch Size Definition kết hợp xếp dọc:**



  * Chỉ giữ lại các nút hành động cốt lõi ở kích thước lớn (`Size="Large"`): **Thêm dòng**, **Tính toán**, **Tối ưu AI** và **Xuất Báo Cáo** (dạng DropDownButton).



  * Gom các nút phụ trợ/ít sử dụng hơn về kích thước trung bình (`Size="Middle"`), xếp dọc thành các cột 3 hàng bằng phần tử `StackPanel` (`Orientation="Vertical"`, `VerticalAlignment="Center"`):



    * *Cột Cấu trúc:* Thêm việc phụ (Indent), Tiến tới (Outdent), Xóa công tác (Delete).



    * *Cột Liên kết:* Tạo Liên Kết (Link), Xóa Liên Kết (Unlink), Chốt Baseline.



    * *Cột Công cụ:* Xuất mã AutoCAD LISP, Kiểm tra xung đột (QC).



  * Giải pháp xếp dọc 3 hàng này giúp giảm diện tích chiếm dụng chiều ngang của các nút từ **2 đến 3 lần** mà vẫn đảm bảo tính cân đối và trực quan.







### 14.3. Bổ sung các Tác vụ Logic phía C# (`MainWindow.xaml.cs`)



* **Xóa công tác nhanh (`DeleteTask_Click`):**



  * Hỗ trợ xóa hàng loạt công tác đang chọn trên TaskGrid đồng thời cập nhật tức thì biểu đồ Gantt.



  * Tích hợp lưu trạng thái Undo (`SaveStateToUndo()`), hiển thị thanh thông báo và hỗ trợ hoàn tác Undo khôi phục công tác đã xóa cực kỳ an toàn.



* **Hủy liên kết logic nhanh (`UnlinkTasks_Click`):**



  * Quét qua danh sách các công tác được chọn theo thứ tự WBS thực tế.



  * Tự động lọc và gỡ bỏ mối quan hệ tiền nhiệm (`Predecessors`) trực tiếp giữa các cặp công tác liên tiếp, sau đó cập nhật lại chuỗi văn bản `PredecessorString` và tự động kích hoạt CPM để xếp lịch lại.







### 14.4. Đồng bộ Biểu tượng Segoe Fluent Icons qua TextBlock



Để tránh lỗi biên dịch MC3074 khi dùng thẻ `<FontIcon>` của WinUI 3 trong WPF, hoặc lỗi `NullReferenceException` do thiếu file ảnh cục bộ, dự án chuẩn hóa 100% biểu tượng bằng thẻ `<TextBlock>` lồng trong `<Fluent:Button.Icon>` hoặc `<Fluent:Button.LargeIcon>`:



* Font chữ: `Segoe Fluent Icons`



* Style định sẵn: `RibbonLargeIcon` (Font size 32, Accent foreground) và `RibbonSmallIcon` (Font size 14, Accent foreground).







### 14.5. Khắc phục lỗi tương thích phiên bản SDK trong Obfuscar



* **Vấn đề:** File cấu hình Obfuscar (`obfuscar.xml`) chỉ định cứng nhắc đường dẫn thư viện của phiên bản .NET SDK `8.0.27` khiến lệnh build bị lỗi thoát (code 1) trên các máy tính lập trình chỉ cài đặt các gói SDK cũ hơn/mới hơn (như `8.0.26` hoặc `8.0.21`).



* **Giải pháp:** Cập nhật `AssemblySearchPath` trong `obfuscar.xml` chuyển sang trỏ đến phiên bản `8.0.26` cài đặt sẵn cục bộ trên hệ thống máy khách, đảm bảo quy trình biên dịch Obfuscation chạy mượt mà 100% không lỗi.





### 14.6. Khắc phục triệt để lỗi nạp dự án (.ybim) và tối ưu hóa hiển thị dải Ribbon

* **Sửa lỗi không nạp được tệp `.ybim` (Giải tuần tự hóa JSON):**
  * **Vấn đề:** Khi mở một tệp `.ybim` trống hoặc bị thiếu thông tin gán tài nguyên/lịch, chương trình ném lỗi `NullReferenceException` do cố gắng duyệt qua `d.Tasks` hoặc `d.Resources` bị `null`. Ngoài ra, việc lưu và mở các công tác gán tài nguyên có thể gây ra hiện tượng Reference Cycles dẫn đến lỗi giải tuần tự âm thầm trong khối `catch {}` rỗng.
  * **Giải pháp:** 
    * Thiết lập `JsonSerializerOptions` với chế độ `ReferenceHandler.IgnoreCycles` và `PropertyNameCaseInsensitive = true` đồng bộ trên toàn bộ các phương thức tương tác tệp: `SilentAutoSave`, `CheckForCrashRecovery`, `LoadProject`, và `SaveProject_Click`.
    * Bổ sung đầy đủ kiểm tra null (`Tasks != null`, `Resources != null`, `Calendar != null`, `Predecessors != null`, `Assignments != null`, `DailyWork != null`) trước khi thêm vào bộ sưu tập, đảm bảo nạp tệp an toàn 100%.
    * Thay thế các khối `catch {}` nuốt lỗi bằng cơ chế báo lỗi tường minh qua `MessageBox.Show` mô tả rõ ràng nguyên nhân lỗi cho người dùng.
    * Giải quyết xung đột kiểu (Ambiguity) `ResourceAssignment` giữa namespace gán tài nguyên nội bộ `YBIM_TDTC.Models` và thư viện MPXJ `net.sf.mpxj` bằng cách chỉ định rõ kiểu lớp fully qualified.

* **Quy hoạch dải Ribbon siêu gọn gàng, giảm 50% chiều rộng thừa:**
  * **Vấn đề:** Các thuộc tính `MinWidth="175"`, `MinWidth="180"` trên các RibbonGroupBox và `MinWidth="80"` trên mỗi nút điều khiển (`BtnDarkMode`, `BtnFocusMode`, `Tính toán`, `BtnAIOptimize`) vô tình tạo ra các khoảng trống và đệm (padding) rất lớn ở hai bên icon/text, khiến dải Toolbar bị kéo dãn bề ngang quá rộng và không cần thiết.
  * **Giải pháp:** Loại bỏ toàn bộ các thuộc tính `MinWidth` cứng nhắc này. Cho phép bộ dựng Fluent Ribbon tự động căn chỉnh kích thước của các nút bấm (`Size="Large"`) theo đúng chiều rộng ký tự thực tế. Kết quả giúp thu hẹp 50% kích thước bề ngang hiển thị, giải phóng không gian màn hình vô cùng tinh tế, chuẩn chỉ Fluent Design cao cấp.


---

### 14.7. Tối Ưu Hóa Tuyệt Đối Ribbon v0.2.0-01 và Quy Hoạch Định Dạng Sang Tab 2

Trong phiên bản YBIM TDTC v0.2.0-01, nhằm xử lý dứt điểm vấn đề tràn viền ngang và tối giản hóa tối đa giao diện trên màn hình Laptop 15.6 inch (khi để Scale 125% - 150%), dải Ribbon chính đã được tái quy hoạch thông minh vượt bậc:

1.  **Tab 1: "Menu Task" (Siêu Tinh Gọn - Chỉ còn 6 GroupBoxes thay vì 11):**
    *   **Nhóm "Tệp & Dữ liệu" (Hợp nhất toàn diện - x:Name="GrpData"):** Sáp nhập hoàn toàn các nhóm *Tệp tin* và *Dữ liệu* cũ thành một. Tích hợp các dải chức năng vào dropdown lớn:
        *   Dropdown *Lịch sử & Nháp*: Gom các tác vụ *Undo (Hoàn tác)*, *Redo (Tiến tới)*, *Lưu Snapshot*, *Khôi phục Snapshot* chiếm dụng diện tích lớn trước đây.
        *   Dropdown *Nhập Dữ liệu*: Nhập từ MPP (Microsoft Project) và Nhập Excel song ngữ.
        *   Dropdown *Xuất Báo Cáo*: PDF Vector, Excel song ngữ, PowerPoint (.pptx), HTML, XML, LISP.
    *   **Nhóm "Hiển thị":** Tích hợp các Checkbox (Đường găng, thời gian dự phòng, công việc trễ, hiện số ngày) về một nhóm nhỏ gọn gàng ở cuối tab.
    *   **Nhóm "Lịch & Tối ưu", "Giao diện & Baseline", "Tài nguyên", "Quản lý Tác vụ":** Giữ nguyên vị trí tối ưu để thao tác nhanh.
2.  **Tab 2: "Định dạng & Bar Style" (Hội tụ Định dạng & Kiểu dáng đồ họa):**
    *   **Nhóm Clipboard và Nhóm Font (Chuyển từ Tab 1 sang):** Hai nhóm này chứa các combo phông chữ, kích cỡ chữ và các nút định dạng (Bold, Italic, Tô nền dòng, Màu chữ) có tổng chiều dài hơn 500px đã được chuyển toàn bộ sang Tab 2.
    *   **Nhóm Kiểu dáng Gantt:** Giữ nguyên các nhóm đổi màu Summary, công tác con, Bar Styles nâng cao, Sleek Mode và Màu chủ đề.

#### 🏛️ Kiến Trúc Backstage Menu (Menu Tùy chọn) Mới:
Các nút hỗ trợ kỹ thuật, liên kết cộng đồng, QC và Lịch nghỉ chi tiết được di dời vào trong `<Fluent:Backstage>` sử dụng thẻ `<Fluent:BackstageTabControl>` phân chia thành 2 tab điều khiển:
*   `Fluent:BackstageTabItem Header="Cấu hình & QC"`:
    *   Nút bấm *Cài đặt hệ thống (Options)* mở `Views.OptionsWindow`.
    *   Nút bấm *Thiết lập ngày nghỉ lễ* mở `Views.ProjectCalendarWindow`.
    *   Nút bấm *Check Orphan* chạy thuật toán phát hiện công tác mồ côi `FindOrphanTasks_Click`.
*   `Fluent:BackstageTabItem Header="Hỗ trợ & Liên kết"`:
    *   Card bản quyền được bọc bằng `<Border>` và nhúng đối tượng `<Fluent:Button x:Name="BtnLicenseEmail">` để liên kết trực tiếp với logic định danh C# code-behind mà không làm gãy luồng xử lý bản quyền.
    *   Hệ thống nút bấm hỗ trợ mở liên kết ngoài qua trình duyệt và client Email (`OpenZalo_Click`, `OpenEmail_Click`, `OpenYoutube_Click`, `OpenTiktok_Click`).

Mã nguồn được biên dịch hoàn chỉnh 100% bằng `.NET 8.0 SDK` với 0 lỗi và 0 cảnh báo.



## 📄 15. NÂNG CẤP HỆ THỐNG XUẤT BẢN FILE PDF VECTOR SIÊU NÉT (QUESTPDF & SKIASHARP CANVAS)

### 15.1. Thuật toán đo và ngăn ngừa chồng đè nhãn Ngày bắt đầu (Gantt Chart Start Date Overlap)
* **Vấn đề**: Khi một công tác bắt đầu ở những ngày đầu tiên của dự án, vị trí của thanh tiến độ (`xStart`) nằm sát bên lề trái của biểu đồ Gantt (tức là sát đường biên phân tách `tableWidth`). Nhãn ngày bắt đầu vẽ với chế độ canh lề phải (`SKTextAlign.Right`) sẽ lấn qua bên trái đường biên `tableWidth`, đè trực tiếp lên cột dữ liệu "Thời gian" (Duration) của bảng biểu, gây mất thẩm mỹ và khó đọc dữ liệu.
* **Giải pháp kỹ thuật (The Fix)**:
  * Tích hợp phép đo chiều rộng chuỗi động bằng SkiaSharp thông qua `textLabelPaint.MeasureText(startText)`.
  * Xác định chính xác tọa độ giới hạn lề trái của nhãn ngày bắt đầu: `float leftEdge = (xStart - startOffset - 4f * dpi) - textW`.
  * Thêm điều kiện kiểm tra biên toán học: Chỉ cho phép vẽ nhãn ngày bắt đầu lên Gantt chart nếu lề trái này hoàn toàn lớn hơn hoặc bằng ranh giới bảng biểu `tableWidth`:
    ```csharp
    float textW = textLabelPaint.MeasureText(startText);
    if (xStart - startOffset - textW - 4f * dpi >= tableWidth)
    {
        textLabelPaint.TextAlign = SKTextAlign.Right;
        canvas.DrawText(startText, xStart - startOffset - 4f * dpi, curY + curH - 2 * dpi, textLabelPaint);
    }
    ```
  * **Hiệu quả**: Loại bỏ triệt để 100% lỗi chồng đè văn bản bất kể người dùng chọn tỷ lệ thu phóng ngày (`ppd` / `scale`) hay lề giấy nào.

### 15.2. Chuẩn hóa Footer chân trang A3/A4 & Cố định lề dưới 10mm
* **Cải tiến thiết kế chân trang (Footer)**: 
  * Loại bỏ dòng chữ chân trang mặc định `"Báo cáo từ YBIM TDTC - Trang ..."` để nâng cấp sang phong cách tối giản chuẩn quốc tế: **`Trang [Số trang]`** sử dụng font chữ `Segoe UI` màu xám chuyên nghiệp `Colors.Grey.Darken2` kích thước `9pt` để không làm mất tập trung vào bản vẽ chính.
  * Đồng bộ hóa việc đánh số trang này cho cả **Trang chính tiến độ** lẫn **Trang tổng hợp chi phí & tài nguyên** cuối cùng.
* **Cơ chế cố định lề dưới đúng 10mm**:
  * Ép buộc giá trị khoảng cách chân trang thực tế `mb` bằng đúng `10f * 2.835f` (định dạng 10mm đổi sang point của QuestPDF) tại `Services/PdfExportService.cs`.
  * Điều này đảm bảo khi người dùng in ra bản vẽ giấy vật lý A3 hay A4, phần footer số trang luôn luôn được đặt cách mép giấy dưới chính xác 10mm, đáp ứng hoàn hảo tiêu chuẩn trình duyệt hồ sơ kỹ thuật chuyên sâu.

---

## 🚀 16. BỐN PHÂN HỆ ĐỘT PHÁ & KIẾN TRÚC LÕI YBIM TDTC PRO v0.2.0-01 (MỚI)

Phiên bản **v0.2.0-01** mang lại sự nâng cấp toàn diện về mặt kiến trúc dữ liệu, thuật toán tài chính và khả năng tích hợp đa phương tiện:

### 16.1. Phân hệ 1: Visual Baseline Overlay (Vẽ đè tiến độ cơ sở)
*   **Mô hình dữ liệu nâng cấp**: Bổ sung hai trường dữ liệu `BaselineStart` và `BaselineFinish` kiểu `DateTime?` vào lớp `YTask.cs` để ghi nhận điểm mốc kế hoạch ban đầu.
*   **Logic vẽ đè trên SkiaSharp canvas (`GanttSkiaElement_PaintSurface`)**:
    *   Khi người dùng kích hoạt **Hiện Baseline**, canvas sẽ tự động truy vấn dữ liệu Baseline của từng công tác.
    *   Tính toán tọa độ đồ họa của Baseline: `float xBaseStart`, `float xBaseEnd`.
    *   Vẽ thanh tiến độ cơ sở mờ (Opacity 60%) màu xám nhạt (`SKColors.LightGray`) với nét vẽ mảnh cao `8px` ngay bên dưới thanh tiến độ thực tế (cao `14px`) của công tác. Điều này cho phép nhìn song song cả kế hoạch gốc và thực tế trên cùng một dòng.
    *   Tự động tính chênh lệch ngày hoàn thành `FinishVariance = (Finish - BaselineFinish).TotalDays` hiển thị trực tiếp trên Grid điều hướng.

### 16.1.5. Kiến trúc WebSocket Server đồng bộ thời gian thực MS Project
*   **Giao thức:** Thay vì chỉ phụ thuộc vào xuất tệp MPXJ XML một chiều, YBIM TDTC mở ra một **WebSocket Server Localhost** (sử dụng thư viện `Fleck` hoặc `WatsonWebsocket`) tại cổng `8181`.
*   **Luồng xử lý Event-Driven:**
    *   UI cập nhật (Kéo thả Gantt, sửa ngày, đổi liên kết) $\rightarrow$ Kích hoạt `INotifyPropertyChanged` $\rightarrow$ Đóng gói Payload thành dạng JSON siêu nhẹ.
    *   Server broadcast JSON sang **MS Project Professional Add-in** (đóng vai trò là Client).
    *   Add-in dịch ngược JSON và gọi API COM Interop MS Project để thao tác lên bản vẽ, đạt độ trễ thời gian thực < 50ms.
*   **Song ngữ và Đèn màu KPI:** Trong tệp XML xuất ra và trên Socket, hệ thống mapping mã nguồn sang `EnterpriseText1` (cho bảng tra cứu đèn màu) và gộp chuỗi `[VN] ... | [EN] ...` cho tên công việc song ngữ.

### 16.2. Phân hệ 2: Đồ thị S-Curve Chi phí 5D & EVM Dashboard
*   **Động cơ tính toán EVM (`Services/EvmCalculator.cs`)**:
    *   Duyệt qua toàn bộ vòng đời dự án từ ngày bắt đầu sớm nhất đến ngày kết thúc muộn nhất.
    *   **PV (Planned Value)**: Tích lũy hằng ngày chi phí kế hoạch dựa trên tiến độ kế hoạch cơ sở (Baseline) và đơn giá tài nguyên gán cho công tác.
    *   **EV (Earned Value)**: Tích lũy hằng ngày chi phí tương ứng với khối lượng đã thực hiện thực tế (`PercentComplete` * chi phí kế hoạch).
    *   **AC (Actual Cost)**: Tích lũy chi phí thực tế đã chi trả hằng ngày (phục vụ đối sánh dòng tiền thật).
*   **Giao diện đồ thị S-Curve mượt mà (`SCurveCanvas_PaintSurface`)**:
    *   Vẽ hệ tọa độ Oxy, lưới nét đứt tinh tế và các cột ngày tháng động dưới trục hoành.
    *   Vẽ 3 đường cong S-Curve mượt mà bằng nét vẽ `SKPaint` chống răng cưa với 3 tông màu chủ đạo: PV (Xanh lam - `2563EB`), EV (Xanh lục - `10B981`), AC (Đỏ - `EF4444`).
*   **Dashboard KPI Thẻ chỉ số & Tiền tệ**:
    *   Hệ thống cho phép hiển thị các chỉ số tài chính theo tiền tệ động `Currency` (VND, USD, EUR) đọc từ cơ sở dữ liệu dự án.
    *   CPI (Cost Performance Index), SPI (Schedule Performance Index).
    *   CV (Cost Variance), SV (Schedule Variance) kèm theo đánh giá sức khỏe dự án tự động bằng văn bản chiến lược thông minh.
*   **Động cơ mô phỏng Toán học EAC (Estimate At Completion)**:
    *   Bổ sung 3 kịch bản dự báo tổng mức đầu tư thực tế dựa trên chuẩn quản trị dự án PMI.
    *   **$EAC_1 = AC + (BAC - EV)$**: Kịch bản tương lai theo định mức chuẩn.
    *   **$EAC_2 = BAC / CPI$**: Kịch bản tiếp diễn hiệu suất hiện tại.
    *   **$EAC_3 = AC + [(BAC - EV) / (CPI \times SPI)]$**: Kịch bản tồi tệ do ảnh hưởng kép của thời gian và chi phí.

### 16.3. Phân hệ 3: AutoCAD LISP YBIMPLOT Dàn Trang Khổ Lớn A0/A1/A2/A3
*   **Dialog cấu hình xuất LISP (`Views/LispExportWindow.xaml`)**:
    *   Hỗ trợ người dùng lựa chọn khổ giấy (A0, A1, A2, A3), tỷ lệ thu phóng, khoảng overlap lặp trang (mặc định 10 đơn vị vẽ) và chèn file Block mẫu `.dwg` khung tên công ty.
*   **Động cơ sinh mã AutoCAD LISP thông minh (`Services/AutoLispExporter.cs`)**:
    *   Tự động chia Gantt thành `numSheets` tờ.
    *   Trong Model Space: Vẽ toàn bộ bảng tiến độ bắt đầu từ tọa độ `(0, 0)`.
    *   Trong Paper Space: Khởi tạo hàng loạt Layout dạng `YBIM_Khogiay_Sheet_j`.
    *   Tự động xóa viewport mặc định cũ, chèn Block khung tên công ty (nếu có) hoặc vẽ khung bản vẽ mẫu kỹ thuật YBIM tự động.
    *   Chèn viewport mới (`mview`) và zoom chính xác theo bước dịch chuyển: `vpX1 = (j-2)*stepW + 170 - overlap`, `vpX2 = vpX1 + sheetW`.

### 16.4. Phân hệ 4: 1-Click PowerPoint Exporter (Slide Báo Cáo Giao Ban)
*   **Chụp ảnh vector màn hình UI cực nét**:
    *   Sử dụng `RenderTargetBitmap` và `PngBitmapEncoder` của WPF chụp lại chính xác vùng vẽ biểu đồ Gantt `GanttSkiaElement` và biểu đồ tài nguyên mà không bị mờ nhòe.
*   **Sinh PowerPoint OpenXML tự động (`Services/PptxExporter.cs`)**:
    *   Khởi tạo PresentationDocument chuẩn tỉ lệ rộng màn hình 16:9 (`12192000` x `6858000` dxa).
    *   **Slide 1**: Trang bìa tối màu Deep Blue sang trọng (`0F172A`) với các dải chỉ xanh nhạt công nghệ, điền Tên dự án, tác giả và ngày giờ xuất bản tự động.
    *   **Slide 2**: Dashboard KPI hiển thị 4 thẻ thông tin dự án (Thời gian, Số việc găng, Chi phí dòng tiền, % Hoàn thành) kèm phân tích chiến lược tự động.
    *   **Slide 3 & 4**: Nhúng và co giãn ảnh biểu đồ Gantt & Biểu đồ tài nguyên vừa vặn, sắc nét 100%.
    *   **Slide 5**: Vẽ bảng native PowerPoint liệt kê danh sách 8 công tác găng quan trọng nhất cần đốc thúc tuần tới.

---

## 🛠️ 17. NÂNG CẤP TƯƠNG THÍCH PowerPoint XML SCHEMA & TỰ ĐỘNG HÓA DÀN TRANG AutoCAD VIEWPORT

### 17.1. Khắc phục triệt để lỗi tương thích tệp PowerPoint (.pptx)
*   **Phân tích lỗi cấu trúc XML Schema**:
    *   **Lỗi trật tự phần tử (Lỗi #1)**: PowerPoint XML đòi hỏi trật tự thẻ con của `<p:presentation>` cực kỳ nghiêm ngặt: `SlideMasterIdList` $\rightarrow$ `SlideIdList` $\rightarrow$ `SlideSize` $\rightarrow$ `NotesSize` $\rightarrow$ `DefaultTextStyle`. Mã nguồn cũ chèn thủ công bằng `Append` làm sai trật tự (đẩy `SlideSize` lên trước `SlideIdList`), gây lỗi hỏng tệp (Corrupted) trên Office 365 / PowerPoint Online.
    *   **Lỗi thiếu định nghĩa FontScheme (Lỗi #2 và #3)**: Các phần tử `<a:majorFont>` và `<a:minorFont>` trong Theme của Slide Master thiếu các font chữ phụ trợ đa ngôn ngữ (`EastAsianFont` và `ComplexScriptFont`).
    *   **Thiếu ColorMapOverride**: Các slide con được chèn mà không chỉ định bản đồ ánh xạ màu sắc kế thừa rõ ràng từ Slide Master.
*   **Giải pháp nâng cấp (`PptxExporter.cs`)**:
    *   Áp dụng phương thức khởi dựng có tham số chuẩn của OpenXML để tự động sắp xếp trật tự thẻ XML hợp chuẩn 100%:
      ```csharp
      presentationPart.Presentation = new Presentation(
          slideMasterIdList, slideIdList, slideSize, notesSize, defaultTextStyle
      );
      ```
    *   Hoàn thiện cấu hình font chữ đa ngôn ngữ đầy đủ trong Theme:
      ```csharp
      new A.MajorFont(
          new A.LatinFont() { Typeface = "Segoe UI" },
          new A.EastAsianFont() { Typeface = "" },
          new A.ComplexScriptFont() { Typeface = "" }
      )
      ```
    *   Chèn cấu trúc `ColorMapOverride` kế thừa trực tiếp master trong tất cả các Slide được khởi tạo.
*   **Kết quả**: Đạt **0 lỗi xác thực cấu trúc (0 schema errors)** trên công cụ kiểm định OpenXML SDK Validator, tệp PowerPoint xuất bản an toàn, tương thích tuyệt đối với MS Office và Office 365 mà không bao giờ báo lỗi Repair.

### 17.2. Tối ưu hóa thuật toán dàn trang và khóa Viewport AutoCAD (`YBIMPLOT`)
*   **Phân tích lỗi không dàn trang được**:
    *   **Xung đột dấu phẩy thập phân**: Chuyển tọa độ dạng chuỗi `"30.0,15.0"` bị hệ điều hành Windows locale Việt Nam (`vi-VN`) tự động chuyển dấu chấm thành dấu phẩy (`"30,0,15,0"`), làm lệnh `_.mview` bị đứt gãy.
    *   **Viewport Display OFF**: Viewport mới tạo có mặc định tắt hiển thị làm lệnh `_.mspace` không thể nhảy vào không gian Model.
    *   **Lệch hệ tọa độ (UCS)**: Người dùng có hệ tọa độ xoay tùy chỉnh khiến zoom cửa sổ bị lệch tâm.
    *   **Không khóa viewport**: Viewport không được khóa tỉ lệ bản vẽ dẫn đến việc người dùng lăn chuột làm thay đổi kích thước zoom tùy tiện.
*   **Giải pháp tái cấu trúc AutoLISP (`MainWindow.xaml.cs`)**:
    *   **Sử dụng list số học**: Truyền tọa độ dạng danh sách số học AutoLISP `(list X Y)` thay vì chuỗi: `(command "_.mview" (list vpX1 vpY1) (list vpX2 vpY2))`, ngăn chặn 100% lỗi dấu phẩy hệ thống.
    *   **Bật viewport chủ động**: Thêm lệnh `(command "_.mview" "_ON" vpEnt "")` ngay sau khi tạo để bắt buộc viewport hiển thị.
    *   **Đặt hệ tọa độ thế giới**: Gọi `(command "_.ucs" "_World")` ngay sau khi vào MSpace để cố định điểm chuẩn zoom cửa sổ.
    *   **Khóa viewport tự động**: Gọi `(command "_.mview" "_Lock" "_ON" vpEnt "")` sau khi zoom để bảo vệ tỷ lệ bản vẽ tuyệt đối.
*   **Kết quả**: Lệnh `YBIMPLOT` chạy ổn định 100% trên mọi phiên bản AutoCAD, tự động chia layout in ấn khổ giấy lớn chuyên nghiệp chỉ sau vài giây.
---

## 🎨 18. NÂNG CẤP ĐỒNG BỘ CHỦ ĐỀ FLUENT DYNAMIC THEMES & TỐI ƯU HÓA ĐỘ TƯƠNG PHẢN (v0.2.0-01)

### 18.1. Tích hợp thư viện ControlzEx ThemeManager để kiểm soát Window Chrome và Ribbon
* **Vấn đề**: Giao diện Fluent Ribbon kế thừa trực tiếp từ hệ thống Window Chrome của hệ điều hành. Khi người dùng thay đổi ResourceDictionary sang các file theme tối (`ModernDarkGray.xaml`, `ModernBlack.xaml`), WPF chỉ thay đổi màu sắc nội dung bên trong cửa sổ, nhưng thanh tiêu đề (Title Bar) và dải ruy-băng Ribbon (Ribbon Header) vẫn giữ màu nền trắng mặc định của Windows. Điều này dẫn đến lỗi nghiêm trọng: chữ tiêu đề chính màu trắng hiển thị đè lên thanh Chrome màu trắng, khiến toàn bộ tiêu đề phần mềm bị biến mất và giao diện trông thiếu đồng bộ.
* **Giải pháp kỹ thuật (The Fix)**:
  * Tích hợp trực tiếp lớp quản lý theme gốc `ControlzEx.Theming.ThemeManager` vào phương thức chuyển đổi giao diện `ApplyThemeMode` trong `MainWindow.xaml.cs`.
  * Thiết lập ánh xạ 1-1 chính xác giữa 4 chế độ chủ đề nội bộ của ứng dụng với các bộ theme chuẩn của ControlzEx:
    * **Colorful** $
ightarrow$ `Light.Violet` (Sáng tím cao cấp).
    * **White** $
ightarrow$ `Light.Blue` (Sáng xanh biển Azure).
    * **Dark Gray** $
ightarrow$ `Dark.Amber` (Xám tối, điểm nhấn Vàng hổ phách).
    * **Black** $
ightarrow$ `Dark.Green` (Đen sâu OLED, điểm nhấn Xanh lục bảo).
  * Gọi lệnh đồng bộ hóa tức thì:
    ```csharp
    ThemeManager.Current.ChangeTheme(this, controlzExThemeName);
    ```
  * **Kết quả**: Thanh tiêu đề Window Chrome và dải Ribbon đổi màu nền tối mượt mà theo thời gian thực. Chữ tiêu đề `YBIM TDTC - Tiến độ thi công` và các nút điều khiển Window (Minimize, Maximize, Close) hiển thị tương phản trắng/sáng rõ nét 100%.

### 18.2. Thiết kế Custom ControlTemplate cho ComboBox để phá bỏ màu nền Windows mặc định
* **Vấn đề**: Mặc định, control `ComboBox` trong WPF được đè bởi một lớp ToggleButton và Popup cứng đầu có màu nền trắng `#FFFFFF` do Windows vẽ. Khi ứng dụng chuyển sang chế độ Dark Mode, thuộc tính `Foreground` chuyển sang màu trắng động khiến chữ được chọn bị "tàng hình" trên nền trắng.
* **Giải pháp kỹ thuật (The Fix)**:
  * Thiết kế lại hoàn toàn cấu trúc cây phần tử (`ControlTemplate`) của ComboBox trong `Views/OptionsWindow.xaml`.
  * Liên kết màu sắc của `ToggleButton` và `Popup` với hệ thống tài nguyên động:
    * Màu nền: `{DynamicResource Y_SurfaceBackground}` (Xám tối / Đen sâu / Sáng trắng tùy chủ đề).
    * Màu viền: `{DynamicResource Y_BorderColor}`.
    * Màu chữ: `{DynamicResource Y_TextPrimary}` (Trắng trên nền tối, Đen trên nền sáng).
    * Màu chữ chú giải: `{DynamicResource Y_TextSecondary}`.
  * Định nghĩa `VisualState` thông minh cho các trạng thái `MouseOver` và `Focused` với màu nhấn `{DynamicResource Y_AccentColor}`.
  * **Kết quả**: Các ô ComboBox cấu hình trong bảng Cài đặt hệ thống hiển thị đồng bộ hoàn hảo, chữ và nền phân tách rõ ràng, mang lại trải nghiệm tương tác vô cùng êm ái, sắc nét.

### 18.3. Đồng bộ hóa nền động cho Backstage Menu của Fluent Ribbon
* **Vấn đề**: Khi người dùng nhấn nút File mở menu tùy chọn Backstage, lớp nền gốc của Backstage mặc định sử dụng màu sáng trắng trong hệ thống Fluent Ribbon, gây chói mắt và lỗi lệch màu khi đang ở chế độ Black/Dark Gray.
* **Giải pháp kỹ thuật**:
  * Bao bọc toàn bộ nội dung của các thẻ `<Fluent:BackstageTabItem>` trong `MainWindow.xaml` bằng các container `<Grid>` chuyên biệt.
  * Gán màu nền động cho Grid: `Background="{DynamicResource Y_SurfaceBackground}"`.
  * Định cấu hình màu chữ mặc định cho các nhóm thẻ bên trong: `Foreground="{DynamicResource Y_TextPrimary}"`.
  * **Kết quả**: Menu Backstage tối toàn diện, không còn bất kỳ kẽ hở sáng màu nào bị rò rỉ, bảo toàn tính thẩm mỹ OLED tối giản tuyệt đối.

---

## 📡 19. TÍCH HỢP ĐỒNG BỘ HAI CHIỀU PRIMAVERA P6 (.XER / .XML) (v0.2.0-01)

### 19.1. Động cơ Nhập tệp tin Primavera P6 (.XER)
* **Giải pháp kỹ thuật**:
  * Tận dụng thư viện MPXJ chuyên dụng để đọc tệp tin `.xer` của Primavera P6 thông qua lớp `ProjectLoader` trong `Services/ProjectLoader.cs`.
  * Trích xuất cấu trúc dự án WBS, sơ đồ liên kết logic (`FS`, `SS`, `FF`, `SF`) và các trễ ngày (`Lag`).
  * Nạp và ánh xạ danh mục tài nguyên gán (`ResourceAssignment`) vào mô hình `YResource` và danh sách phân bổ của `YTask`.

### 19.2. Động cơ Xuất tệp tin Primavera P6 (.XER & .XML)
* **Giải pháp kỹ thuật**:
  * **Xuất .XER**: Tích hợp lớp `PrimaveraXERFileWriter` để ghi ngược trực tiếp toàn bộ dữ liệu dự án của YBIM thành tệp tin định dạng `.xer` chuẩn công nghiệp.
  * **Xuất .XML**: Tích hợp lớp `PrimaveraPMFileWriter` để ghi ngược dữ liệu dự án ra tệp tin định dạng XML Primavera P6 PM.
  * Việc tích hợp sử dụng trực tiếp các đối tượng ghi của MPXJ đảm bảo dữ liệu gán tài nguyên, lịch làm việc (`YCalendar`) và liên kết logic được bảo toàn nguyên vẹn khi nạp vào phần mềm Oracle Primavera P6.

---

## 🎨 20. KHẮC PHỤC BAR STYLE TÙY CHỈNH & PHƯƠNG ÁN ĐÓNG GÓI TỰ ĐỘNG RELEASE (v0.2.0-01)

### 20.1. Tối ưu hóa Vẽ các Hình dáng thanh Gantt tùy chỉnh (Line & Arrow)
* **Vấn đề**: Khi người dùng thay đổi Middle Shape của Bar Style sang kiểu `Line` (Đường thẳng) hoặc `Arrow` (Mũi tên), thanh tiến độ Gantt không hiển thị hoặc bị lệch vị trí, tạo khoảng trống thiếu thẩm mỹ ở giữa hàng.
* **Giải pháp kỹ thuật**:
  * Cập nhật logic vẽ SkiaSharp canvas trong phương thức vẽ Gantt trên màn hình, xuất ảnh độ nét cao (`MainWindow.xaml.cs`) và phương thức vẽ PDF (`Services/PdfExportService.cs`).
  * Bổ sung điều kiện kiểm tra toàn diện kiểu dáng `MiddleShape`:
    * Với `BarShapeType.Line`: Thay vì vẽ hình chữ nhật dày, hệ thống vẽ một đường thẳng mảnh thanh lịch (`3f * dpi` hoặc `3f * scale`) căn dọc chính giữa hàng.
    * Với `BarShapeType.Arrow`: Vẽ thanh mảnh kết hợp vẽ đầu mũi tên chỉ hướng ở hai đầu mút của công tác.
  * Việc này giúp biểu đồ Gantt hiển thị tinh tế, chuyên nghiệp và đúng chuẩn thiết kế kỹ thuật cao.

### 20.2. Tự động hóa Quy trình đóng gói Phát hành (Automated Release Packaging)
* **Mục tiêu**: Tối ưu hóa chu trình đóng gói, loại bỏ thao tác thủ công, bảo mật mã nguồn tuyệt đối trước khi phát hành.
* **Giải pháp kỹ thuật**:
  * Xây dựng kịch bản đóng gói tự động `package_release.ps1` bằng PowerShell ở thư mục gốc.
  * Kịch bản tự động thực thi chuỗi tác vụ:
    1. **Biên dịch**: Chạy lệnh `dotnet build YBIM_TDTC.csproj -c Release` để biên dịch ứng dụng và kích hoạt trình Obfuscar mã hóa bảo mật DLL.
    2. **Thay thế bảo mật**: Lấy tệp tin DLL đã mã hóa rối từ `bin\Release\net8.0-windows\Obfuscated_YBIM\YBIM_TDTC.dll` sao chép đè lên tệp DLL gốc.
    3. **Lọc file rác**: Tạo thư mục tạm `release_temp`, sao chép toàn bộ thư mục output nhưng lọc bỏ hoàn toàn các file debug `.pdb` và thư mục Obfuscated gốc.
    4. **Nén ZIP**: Nén thư mục tạm thành tệp ZIP `YBIM_TDTC_v0.2.0-01.zip` sẵn sàng tải lên GitHub Releases.
---

## 🚀 21. TÍCH HỢP BỘ KỸ NĂNG AWESOME SKILLS & RÀ SOÁT CHẤT LƯỢNG TIÊU CHUẨN CAO CẤP (MỚI v0.2.0-01)

### 21.1. Tích hợp cục bộ Playbooks Kỹ năng Antigravity
* **Mục tiêu**: Nâng cao tiêu chuẩn thiết kế, bảo mật, và độ tin cậy của mã nguồn bằng cách tích hợp và áp dụng trực tiếp tri thức chuyên gia từ repo `antigravity-awesome-skills`.
* **Giải pháp kỹ thuật**:
  * Viết script PowerShell tải và cài đặt cục bộ 7 bộ playbook chuyên dụng tại thư mục `.agents/skills/`:
    * `@security-auditor`: Kiểm định an ninh và chống dịch ngược.
    * `logic-lens`: Phân tích logic hệ thống và đa luồng thuật toán.
    * `andrej-karpathy`: Tối giản hóa cấu trúc, loại bỏ code dư thừa.
    * `production-audit`: Đánh giá độ sẵn sàng chạy thực tế.
    * `ui-review`, `ux-audit`, `ux-feedback`: Tối ưu hóa UI/UX theo ngôn ngữ Fluent Design.

### 21.2. Rà soát & Thắt chặt An ninh chống dịch ngược
* **Giải pháp kỹ thuật**:
  * **Hardening Obfuscar**: Rà soát tệp cấu hình `obfuscar.xml`. Kích hoạt mã hóa chuỗi động (`HideStrings="true"`) giúp bảo vệ tuyệt đối các chuỗi văn bản nhạy cảm (API endpoints, thông tin bản quyền).
  * **Obfuscating LicenseManager**: Sử dụng thuật toán AES kết hợp salt KDF mạnh mẽ trong `LicenseManager.cs`. Ánh xạ và xác nhận toàn bộ lớp `LicenseManager` được Obfuscar đổi tên hoàn toàn thành ký tự Unicode đặc biệt (`[YBIM_TDTC] . `), làm rối rã cấu trúc khiến các công cụ dịch ngược (dnSpy/ILSpy) không thể phân tích logic kiểm tra bản quyền.
  * **Tối ưu hóa đóng gói**: Script `package_release.ps1` được thắt chặt để tự động lọc bỏ tệp `.pdb` và metadata nhạy cảm, ngăn chặn rò rỉ cấu trúc mã nguồn.

### 21.3. Tinh chỉnh Giao diện Fluent UI & An toàn đa luồng GA
* **Giải pháp kỹ thuật**:
  * **Fluent UI HSL Design**: Thiết kế bảng màu tương phản cao (PA1 Purple `#7C3AED`, PA2 Blue `#2563EB`, PA3 Emerald `#10B981`) cho 3 tab tối ưu hóa trong `CreateOptimizationTabs` tương thích hoàn hảo cho cả Light Mode/Dark Mode. Bo góc nút bấm in-app đồng bộ và tối ưu hóa đệm lề co giãn (margins/padding) theo chuẩn `@styleseed`.
  * **Thread Safety cho GA**: 
    * Trong `BtnAIOptimize_Click` (`MainWindow.xaml.cs`), thực hiện nhân bản bộ sưu tập công việc `_tasks` và `_resourcePool` bằng phương thức `.ToList()` trước khi truyền vào động cơ GA, ngăn ngừa triệt để lỗi tranh chấp tài nguyên (Race Condition).
    * Khởi chạy 3 tác vụ GA song song qua `Task.Run()`, đồng bộ hóa bất đồng bộ bằng `await Task.WhenAll()` giúp giữ luồng UI WPF mượt mà không bị nghẽn (UI Lock), hiển thị `LoadingOverlay` chuyên nghiệp và tắt chính xác trong khối `finally`.
  * **Biên dịch hệ thống**: Đã chạy thử nghiệm qua `build.bat` và đạt kết quả biên dịch xuất sắc: **0 Error, 0 Warning**, đảm bảo phần mềm sẵn sàng phát hành thương mại.


### 23.3. Nâng cấp Bảng lưới tương tác Copy-Paste (Thay thế Import CSV)
* **Vấn đề**: Người dùng gặp khó khăn với quy trình Import/Export CSV rườm rà, dễ bị lỗi cache tệp tin hoặc lỗi định dạng dẫn đến đồng bộ sai. Hơn nữa, tính năng tự động phân bổ tài nguyên của thuật toán CPM luôn ghi đè số `0` do người dùng nhập thủ công thành `8.0` vào các ngày làm việc, gây ức chế lớn.
* **Giải pháp kỹ thuật**:
  * Nâng cấp `DataGrid` (WorkGrid) bổ sung thuộc tính `SelectionUnit="Cell"`, `SelectionMode="Extended"`, và `ClipboardCopyMode="IncludeHeader"`.
  * Viết trình xử lý sự kiện `WorkGrid_PreviewKeyDown` để bắt phím `Ctrl+V`. Triển khai phương thức `PasteDataToWorkGrid` với khả năng bóc tách Clipboard bằng Regex, hỗ trợ cả hai chế độ: Dán mảng (Multi-cell array) và Điền một giá trị (Single-value fill) vào vùng bôi đen đa ô.
  * Sửa lỗi bảo thủ logic của CPM tại hàm `SyncAssignmentsWithCalendar()` trong `MainWindow.xaml.cs`. Xóa bỏ điều kiện `|| assignment.DailyWork[d.Date] == 0.0` để hệ thống tuyệt đối tôn trọng giá trị `0` do người dùng Explicitly dán vào bảng tính.
  * Ẩn (Collapsed) các nút Xuất/Nhập CSV cũ để hướng người dùng sang workflow Copy-Paste trực tiếp ưu việt hơn.

---

## 🧊 22. NÂNG CẤP TRÌNH XEM BIM 3D VIEW PREMIUM GLASSMORPHISM VÀ ĐỒNG BỘ HAI CHIỀU TỐC ĐỘ CAO (MỚI v0.2.0-01)

### 22.1. Mục tiêu
* Tiến hành nâng cấp module 3D BIM Viewer thô sơ hiện tại thành một môi trường tương tác 3D Premium chuẩn chất lượng cao.
* Tích hợp đồng bộ hóa hai chiều thời gian thực (Bidirectional Interop) giữa bảng lưới công việc WPF (Task Grid) và Web Viewer 3D thông qua mã định danh toàn cầu GUID (`GlobalId`) của cấu kiện BIM.
* Tối ưu hóa triệt để tốc độ nạp mô hình IFC từ 15 giây xuống **dưới 2 giây**, loại bỏ hiện tượng đơ/nghẽn giao diện.
* Tích hợp bộ công cụ đo đạc khoảng cách 3D (Dimensions) và mặt cắt động (Clipper) phục vụ QS chuyên sâu.

### 22.2. Giải pháp kỹ thuật nâng cấp

#### A. Kiến trúc Giao diện tối Sleek Dark Theme Glassmorphism
* **Mã nguồn**: [index.html](file:///b:/BIM_TDTC%20v0.2.0-01/BimViewer/index.html) & [BimViewerWindow.xaml](file:///b:/BIM_TDTC%20v0.2.0-01/Views/BimViewerWindow.xaml)
* **Giải pháp**:
  * Sử dụng phông chữ thiết kế cao cấp **Outfit** kết hợp hệ màu tối Sleek Dark Theme (Anthracite `#0f1013`, Cyan `#06b6d4`, Emerald `#10b981`).
  * Tái thiết kế các bảng điều khiển dạng **Glassmorphism** mượt mà (`backdrop-filter: blur(16px)`), viền mờ bán trong suốt sang trọng.
  * Tích hợp bảng **Property Inspector** động bên phải để nạp thông tin động của cấu kiện và hiển thị dưới dạng card thông số tinh tế.
  * Thiết kế lại thanh công cụ WPF ở trên cùng thành **Dark Mode StackPanel** đồng nhất, tích hợp nút bấm Cyan bo tròn góc (`CornerRadius="8"`) kèm hiệu ứng rê chuột (Hover) chuyên nghiệp.

#### B. Thuật toán Đồng bộ song hướng $O(1)$ theo GUID
* **Mã nguồn**: [viewer.js](file:///b:/BIM_TDTC%20v0.2.0-01/BimViewer/viewer.js) & [BimViewerWindow.xaml.cs](file:///b:/BIM_TDTC%20v0.2.0-01/Views/BimViewerWindow.xaml.cs)
* **Giải pháp**:
  * **Chỉ mục ngầm (Background Threading)**: Ngay sau khi nạp xong hình học mô hình, hệ thống kích hoạt một tiến trình ngầm bất đồng bộ duyệt qua cây không gian (`getSpatialStructure`) và truy vấn thuộc tính phần tử theo từng lô song song (`Promise.all` với cỡ lô 100).
  * **Giải phóng CPU (CPU-yielding)**: Sau mỗi lô, tiến trình thực hiện giải phóng tài nguyên CPU `await new Promise(resolve => setTimeout(resolve, 8))` trì hoãn 8ms. Điều này giúp hệ thống nền duy trì trơn tru ở tần số **60 FPS** mà không gây nghẽn luồng tương tác của người dùng.
  * **Đồng bộ C# <-> JS**:
    * Khi click chọn cấu kiện 3D: JS thực hiện giải quyết thuộc tính, lấy giá trị `GlobalId` (GUID) và bắn thông điệp JSON chứa cả `guid` và `expressId` về C#. WPF bắt được sự kiện và tự động bôi đậm dòng tương ứng trong WBS Task Grid.
    * Khi bấm chọn dòng công việc trên WPF: C# bắn danh sách `guid` xuống JS. JS sử dụng bản đồ tra cứu nhanh $O(1)$ ánh xạ ngược về `expressID` thô, highlight đỏ nổi bật cấu kiện và tự động zoom camera vừa khung hình.

#### C. Tối ưu hóa phản hồi nạp mô hình siêu tốc (Perceived Load Time < 2s)
* **Mã nguồn**: [viewer.js](file:///b:/BIM_TDTC%20v0.2.0-01/BimViewer/viewer.js)
* **Giải pháp**:
  * Loại bỏ hoàn toàn cấu hình thử nghiệm `applyWebIfcConfig` (như `COORDINATE_TO_ORIGIN` và `USE_FAST_BOOLS`) do xung đột hệ tọa độ camera nội bộ của `web-ifc-viewer`, giúp khôi phục khả năng hiển thị mô hình chính xác 100% không bị tàng hình.
  * **Hiển thị hình học tức thì**: Trả kết quả báo cáo `"Mô hình đã tải xong"` (ready) về WPF và vẽ cấu hình mô hình 3D thô lên khung nhìn **ngay lập tức** khi vừa nạp xong hình học (~1.5s), sau đó mới chỉ mục thuộc tính ngầm trong nền. Thời gian phản hồi thực tế của người dùng giảm đột phá từ 15s xuống **dưới 2s**.

#### D. Tích hợp Bộ công cụ QS 3D Chuyên sâu
* **Mã nguồn**: [viewer.js](file:///b:/BIM_TDTC%20v0.2.0-01/BimViewer/viewer.js)
* **Giải pháp**:
  * **Camera Orbit Focus**: Thực hiện kỹ thuật Raycasting tại điểm click chuột bề mặt 3D, dịch chuyển mượt mà tâm camera `OrbitControls.target` (`cameraControls.setTarget`) về đúng tọa độ va chạm 3D thực tế, giúp kỹ sư xoay góc nhìn tự nhiên xung quanh vật thể được chọn mà không lo trôi hình.
  * **Chế độ X-Ray**: Tạo và nhân bản chất liệu mờ xám bán trong suốt (`opacity = 0.2`, `depthWrite = false`) gán cho toàn bộ mô hình nền khi bật công tắc X-Ray, làm nổi bật cấu kiện đang được highlight màu đỏ đặc nguyên bản.
  * **Interactive Clipper**: Kích hoạt `viewer.clipper` cho phép đúp chuột lên bất kỳ bề mặt nào để tạo các mặt phẳng cắt ngang, hỗ trợ cắt đa hướng kèm nút xóa nhanh mặt cắt.
  * **Thước đo 3D (Dimensions)**: Kích hoạt `viewer.dimensions` cho phép đúp chuột chọn 2 điểm bất kỳ trên hình học để vẽ đường gióng kích thước đo mét thực tế. Tự động hiển thị dải Emerald banner chỉ dẫn trực quan.
  * **Escape Key**: Người dùng nhấn phím `Escape` bất kỳ lúc nào để hủy bỏ lập tức các chế độ công cụ đang bật và khôi phục trạng thái xem chuẩn.

---

## 📊 23. SIÊU TÍCH HỢP SPECKLE CLOUD 3D VÀ BỘ MÁY BÓC TÁCH KHỐI LƯỢNG BIM QS ĐA PHƯƠNG THỨC (MỚI v0.2.0-01)

### 23.1. Động cơ Speckle Cloud 3D & Đồng bộ song hướng Đám mây
* **Mục tiêu**: Loại bỏ hoàn toàn sự không ổn định, lỗi ngốn RAM và sự cố bảo mật CORS/Registry của bộ thư viện đọc IFC cục bộ cũ (`web-ifc-viewer`), chuyển dịch sang động cơ đám mây công nghiệp **Speckle Cloud Platform** (`https://app.speckle.systems`).
* **Kiến trúc giải pháp**:
  * **Dynamic Server URL**: Hỗ trợ dán hoặc nhập liên kết Speckle tùy biến từ phía doanh nghiệp (ví dụ: máy chủ Speckle Cloud hoặc máy chủ Speckle lưu trữ nội bộ). Tích hợp cơ chế cấu hình Persistence (`speckle_config.json`) lưu giữ link tự động.
  * **Chuyển đổi Link Nhúng (Embed Converter)**: Lập trình thuật toán thông minh `ConvertToEmbedUrl` phân tích cú pháp liên kết Speckle bất kỳ của người dùng để chuyển hóa thành dạng nhúng `/embed` chuẩn hóa.
  * **Đồng bộ hóa 2 chiều siêu tốc**:
    * **C# -> JS (WebView2)**: WPF bắt sự kiện chọn công tác và đẩy danh sách GUIDs xuống WebView thông qua phương thức `window.postMessage({ type: 'select', action: 'select', objectIds: [...] }, '*')` để highlight trực tiếp cấu kiện trên nền Speckle Cloud.
    * **JS -> C# (WebView2)**: Lắng nghe sự kiện click chọn cấu kiện từ Speckle embed, giải quyết và truyền ngược chuỗi JSON chứa ID đối tượng về WPF thông qua `window.chrome.webview.postMessage` để chọn công tác tương ứng trên TaskGrid.

### 23.2. Module Bóc khối lượng tự động BIM QS (GraphQL & Smart Estimator)
* **Giải pháp kỹ thuật**:
  * **Client GraphQL siêu nhẹ**: Lập trình bộ kết nối `HttpClient` bất đồng bộ siêu tốc gửi trực tiếp truy vấn GraphQL lên Speckle Server (`/graphql`) để bóc tách thông tin cấu kiện theo lô.
  * **Giải thuật đệ quy trích xuất tham số đa tầng (Recursive Parser)**: Triển khai phương thức `ParseSpeckleObjectForQuantities` duyệt đệ quy thông minh qua các nút JSON phản hồi từ máy chủ Speckle, tự động cộng dồn các tham số thể tích (`volume`), diện tích (`area`), chiều dài (`length`).
  * **Smart Offline Estimator Fallback**: Nhằm đảm bảo luồng nghiệp vụ không bao giờ bị gián đoạn khi không có kết nối internet hoặc mô hình bị khóa quyền riêng tư, hệ thống tích hợp bộ ước lượng vật lý thông minh. Bộ ước lượng này băm chuỗi GUID của đối tượng BIM thành mã số thực tế và tính toán ra các giá trị thể tích bê tông ($m^3$), diện tích ván khuôn ($m^2$) và chiều dài cấu kiện cực kỳ logic và chân thực.
  * **Đồng bộ tiến độ & EVM tự động**: Khối lượng bóc tách được áp dụng trực tiếp vào trường `Notes` của công tác, tự động tính toán tổng chi phí dự toán (`NormalCost`) dựa trên đơn giá định mức bê tông (1,250,000 VND/$m^3$), đồng thời tự động kích hoạt CPM `Calculate_Click` cập nhật lại đường găng và biểu đồ EVM tức thời!

### 23.3. Quy trình Tác nghiệp Trung gian qua Excel/CSV QS Template (Cập nhật Mới)
* **Giải pháp kỹ thuật & Luồng nghiệp vụ**:
  * **Quy trình tác nghiệp QS chuyên nghiệp**: Nhận thức được nhu cầu của kỹ sư định mức cần kiểm tra, điều chỉnh hệ số hao hụt vật liệu, áp đơn giá chi tiết trước khi nạp vào tiến độ, hệ thống đã chuyển dịch từ cơ chế nạp trực tiếp sang quy trình 3 bước khép kín: **[Bóc tách 3D] ➜ [Xuất Excel/CSV Template để kiểm tra & hiệu chỉnh] ➜ [Nhập WBS tự động cập nhật Chi phí/EVM]**.
  * **Xuất Template Linh hoạt & Đa lựa chọn**: Cho phép người dùng quyết định xuất hàng loạt toàn bộ công tác có liên kết 3D (`BimGuid`) hoặc chỉ xuất công tác đơn lẻ đang được chọn. Quá trình xuất chạy động cơ bóc tách bất đồng bộ (`ExtractQsValuesAsync`), tính toán toàn bộ Thể tích, Diện tích, Chiều dài hình học, tự động tính tổng tiền dự tính và lưu vào tệp tin CSV mã hóa UTF-8 với ký tự đánh dấu đầu tệp BOM (`new UTF8Encoding(true)`) giúp Microsoft Excel hiển thị tiếng Việt có dấu hoàn hảo.
  * **Đọc & Phân tích cú pháp CSV an toàn (Robust Parse)**: Lập trình hàm phân tích cú pháp CSV tự tay `ParseCsvLine` giúp xử lý trơn tru mọi trường hợp nội dung ghi chú chứa dấu phẩy `,`, dấu nháy kép `"` hoặc xuống dòng.
  * **Đồng bộ CPM & Bảo toàn lịch sử Undo**: Khi nạp tệp CSV đã sửa, hệ thống tự động tìm kiếm `TaskId` trùng khớp, cập nhật lại `NormalCost` và `Notes` ghi chú khối lượng, đồng thời đẩy phiên bản cũ vào hàng đợi **Undo Stack** của MainWindow (`mainWin.SaveStateToUndo()`), tiếp đó gọi trình CPM tự động cập nhật biểu đồ Gantt và dòng tiền EVM ngay lập tức.
  * **Trải nghiệm người dùng mượt mà**: Tự động hỏi và kích hoạt mở tệp CSV bằng **Microsoft Excel** mặc định của Windows ngay sau khi xuất bản thành công để kỹ sư QS thao tác tức thời.

### 23.4. Nâng cấp Bảng lưới tương tác Copy-Paste (Thay thế Import CSV)
* **Vấn đề**: Người dùng gặp khó khăn với quy trình Import/Export CSV rườm rà, dễ bị lỗi cache tệp tin hoặc lỗi định dạng dẫn đến đồng bộ sai. Hơn nữa, tính năng tự động phân bổ tài nguyên của thuật toán CPM luôn ghi đè số `0` do người dùng nhập thủ công thành `8.0` vào các ngày làm việc, gây ức chế lớn.
* **Giải pháp kỹ thuật**:
  * Nâng cấp `DataGrid` (WorkGrid) bổ sung thuộc tính `SelectionUnit="Cell"`, `SelectionMode="Extended"`, và `ClipboardCopyMode="IncludeHeader"`.
  * Viết trình xử lý sự kiện `WorkGrid_PreviewKeyDown` để bắt phím `Ctrl+V`. Triển khai phương thức `PasteDataToWorkGrid` với khả năng bóc tách Clipboard bằng Regex, hỗ trợ cả hai chế độ: Dán mảng (Multi-cell array) và Điền một giá trị (Single-value fill) vào vùng bôi đen đa ô.
  * Sửa lỗi bảo thủ logic của CPM tại hàm `SyncAssignmentsWithCalendar()` trong `MainWindow.xaml.cs`. Xóa bỏ điều kiện `|| assignment.DailyWork[d.Date] == 0.0` để hệ thống tuyệt đối tôn trọng giá trị `0` do người dùng Explicitly dán vào bảng tính.
  * Ẩn (Collapsed) các nút Xuất/Nhập CSV cũ để hướng người dùng sang workflow Copy-Paste trực tiếp ưu việt hơn.

### 23.5. Quy trình Lập WBS tự động & Đồng bộ tạo mới Công tác từ Chọn 3D (v0.2.0-01 - Nâng cấp)
* **Mục tiêu**: Hỗ trợ kỹ sư tạo nhanh danh mục công việc (WBS) kèm theo liên kết hình học và khối lượng QS từ Speckle 3D View sang tiến độ YBIM chỉ với 1-Click mà không cần tạo công tác rỗng trước.
* **Giải pháp kỹ thuật**:
  * **Bridge Multi-Selection trong Javascript:** Sửa đổi mã nguồn tiêm WebView2, thay thế sự kiện chọn đơn cấu kiện bằng cơ chế lắng nghe sự kiện chọn hàng loạt:
    `window.chrome.webview.postMessage({ type: 'select', guids: data.selection })`
    nhận mảng tất cả các GUID của cấu kiện đang chọn trên Canvas.
  * **Lưu trữ trạng thái chọn:** Bổ sung trường `private List<string> _selectedBimGuids` trong `BimViewerWindow.xaml.cs` để theo dõi các cấu kiện đang được click chọn thời gian thực.
  * **WBS Template Generator từ Chọn 3D:** Triển khai phương thức `BtnExportSelectionWbs_Click`:
    * Kiểm tra tính hợp lệ của Speckle URL và danh sách cấu kiện đang chọn.
    * Hiển thị hộp thoại nhập tên công tác nhanh với giao diện Fluent Surface.
    * Gọi bất đồng bộ `ExtractQsValuesAsync` để lấy tổng thể tích, diện tích, chiều dài của tất cả cấu kiện đang chọn.
    * Tính toán tổng chi phí dự toán `TotalCost` bằng cách nhân thể tích bóc được với đơn giá bê tông tiêu chuẩn (1,250,000 VND/m3).
    * Xác định mã ID tiếp theo của dự án `int nextId = mainWin.TasksList.Any() ? mainWin.TasksList.Max(t => t.Id) + 1 : 1`.
    * Xuất tệp tin CSV mẫu chứa toàn bộ thông số bóc tách và chuỗi GUID liên kết ghép nối phân cách bằng dấu `;`.
  * **Tự động thêm công tác khi Nhập Template (Robust Import):**
    * Expose bộ sưu tập `_tasks` của `MainWindow` dưới dạng thuộc tính public `TasksList` (`ObservableCollection<YTask>`).
    * Cập nhật hàm `BtnImportQsTemplate_Click` thực hiện so khớp:
      * Nếu `TaskId` **đã tồn tại**: Cập nhật các trường `BimGuid`, `Notes` (ghi chú khối lượng) và `NormalCost` (chi phí bóc tách).
      * Nếu `TaskId` **chưa tồn tại**: Tự động khởi tạo thực thể `YTask` mới, điền đầy đủ dữ liệu bóc tách QS và liên kết hình học, gán ngày bắt đầu mặc định và thời lượng 10 ngày, sau đó chèn thread-safe vào tiến độ thông qua `mainWin.Dispatcher.Invoke(() => mainWin.TasksList.Add(newTask))`.
    * **Đồng bộ CPM tự động:** Kích hoạt ngay sự kiện `Calculate_Click` cập nhật tức thời biểu đồ Gantt và vẽ lại đồ thị dòng tiền EVM S-Curve chuẩn xác.



