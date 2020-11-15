---
author: TraiOi
title:  "Understand S.M.A.R.T Attribute Data"
date:   2018-11-02
categories: drive
---
Giúp hiểu được các thuộc tính dữ liệu của ổ cứng theo chuẩn S.M.A.R.T <!--more-->

# I. Giới thiệu

 S.M.A.R.T (Self-Monitoring, Analysis, and Reporting Technology) là công nghệ giám sát được hỗ trợ trên hầu hết thiết bị phần cứng hiện nay. Với công nghệ này, các vấn đề internal và external của ổ đĩa sẽ được giám sát lại và báo cáo lại cho người dùng tốt nhất.

# II. S.M.A.R.T attributes

Mỗi thuộc tính của SMART gồm nhiều cột được mô tả trong `smarctl -A <device>`

> - **ID:** ID number của thuộc tính.
> - **Name (ATTRIBUTE_NAME):** Tên của thuộc tính.
> - **Value (VALUE):** Thông số hiện tại của thuộc tính. Value càng cao càng tốt hơn (ngoại trừ nhiệt độ của ổ cứng).
> - **Worst (WORST):** Thông số bad của ổ cứng, được ghi lại tại thời điểm mà `smart=on`.
> - **Threshold (THRESH):** Giới hạn cho phép.
> - **Raw value (RAW_VALUE):** Giá trị raw hiện tại, được chuyển đổi thành thông số **Value**.

# III. Known ATA S.M.A.R.T. attributes

| ID | Thuộc tính | ! | Mô tả |
| --- | --- | --- | --- |
| 01 | **Read Error Rate** | | Tỉ lệ lỗi đọc phần cứng xảy ra khi đọc dữ liệu từ bề mặt đĩa. Các giá trị khác 0 có nghĩa là có vấn đề với đĩa, đầu đọc/ghi (bao gồm cả vết nứt, đầu bị hỏng, kết nối kém với apdapter/module, ..). Giá trị **Value** càng cao thì khả năng hư ổ đĩa càng cao.   |
| 02 | **Throughput Performance** | | Tham số cho biết hiệu suất thoughput của ổ cứng. Giá trị **Value** càng giảm thì khả năng cao ổ cứng gặp vấn đề càng cao. |
| 03 | **Spin-Up Time** | | Thời gian trung bình (đơn vị ms hoặc s) quay của ổ đĩa (spinup) được tính từ 0 RPM đến thời thời điểm hiện tại. **Raw value** càng cao tương ứng với **Value** càng thấp thì tốc độ quay của ổ đĩa càng chậm, khả năng cao ổ đĩa đang gặp vấn đề. |
| 04 | **Start/Stop Count** | | Tham số **Raw value** cho biết số lần (chu kì) spindle (trục chính) của ổ cứng bắt đầu quay và dừng quay. Spindle được bật lên và bắt đầu quay khi ổ đĩa được bật (tương tác với nguồn điện, thao tác mở máy, ..). Mỗi ổ đĩa có giới hạn chu kì này do nhà sản xuất quy định. **Value** (đơn vị %) càng giảm vượt qua **Threshold** chứng tỏ hiệu suất của ổ cứng đang gặp vấn đề. |
| 05 | **Reallocated Sectors Count** | ![critical](/img/critical.png) | Tham số **Raw value** cho biết số lượng các sectors đã được reallocated. Khi ổ cứng tìm thấy lỗi đọc/ghi (bad blocks), ổ cứng sẽ "đánh dấu" sector này là "reallocated" và chuyển dữ liệu (remapping) lên khu vực riêng biệt dự phòng (reallocated sector), tất cả bad blocks sẽ được giấu trong reallocatted sector này. Tham số **Raw value** càng tăng nghĩa là càng có nhiều sector được reallocated, chứng tỏ ổ cứng đang gặp vấn đề. (Tham số này ảnh hưởng đến tốc độ đọc/ghi của ổ đĩa). <br /> **Value** bắt đầu từ 100 và giảm dần về 0, cho thấy tỉ lệ % còn lại của số reallocated được nhà sản xuất cho phép. |
| 06 | **Read Channel Margin** | | Updating.. |
| 07 | **Seek Error Rate** | | **Raw value** cho biết số lần tìm kiếm (seek) bị lỗi của đầu từ, tham số này càng cao thì khả năng hệ thống cơ học của ổ đĩa đang gặp vấn đề. |
| 08 | **Seek Time Performance** | | **Raw value** cho kết quả của phép tính tìm kiếm của đầu từ tại thời điểm nhất định. **Value** là hiệu suất trung bình của các phép tính tìm tiếm được tính qua các đợt ghi lại của **Raw value**. **Value** càng giảm chứng tỏ hệ thống cơ học của ổ cứng đang gặp vấn đề. |
| 09 | **Power-On Hours** | | **Raw value** hiển thị tổng số giờ (hoặc phút, giây, tuỳ thuộc vào nhà sản xuất) ở trạng thái power-on. <br /> **Value:** Luôn 100. |
| 10 | **Spin Retry Count** | ![critical](/img/critical.png) | **Raw value** cho biết số lần thử lại khi ổ đĩa cố gắng quay khi gặp lỗi nếu lần quay đầu tiên bị thất bại. **Value** cho biết tổng số lần ổ đĩa cố gắng quay cho đến khi đạt được tốc độ quay ổn định. **Value** càng tăng chứng tỏ ổ đĩa đang gặp vấn đề hoặc xuống cấp trầm trọng. |
| 11 | **Recalibration Retries** <br />  <br /> **Calibration Retry Count** | | Khi khởi động máy, kết nối với nguồn điện, .. ổ đĩa sẽ tiến hành hiệu chỉnh đầu từ và các rãnh từ. **Raw value** cho biết số lần ổ đĩa phải hiệu chỉnh lại sau khi lần đầu không thành công tại thời điểm nhất định. **Value** là tổng số lần được tính từ **Raw value**. **Value** càng cao chứng tổ ổ cứng đang gặp vấn đề. |
| 12 | **Power Cycle Count** | | **Raw value** là tổng số chu kỳ bật/tắt hoàn toàn của ổ đĩa. <br /> **Value:** luôn 100. |
| 13 | **Soft Read Error Rate** | | Theo dõi số lỗi ECC (Error Correction Code) có thể sửa được. |
| 22 | **Current Helium Level** | | (Updating ...) |
| 170 | **Available Reserved Space** | | (Liên quan với **5**)  |
| 171 | **SSD Program Fail Count** | | (Kingston) (giống **181**). |
| 172 | **SSD Erase Fail Count** | | (Kingston) (giống **182**). |
| 173 | **SSD Wear Leveling Count** <br /><br />**Wear Leveling Count** | | **Wear Leveling** (độ hao mòn) được quản lý bởi flash, sử dụng thuật toán để sắp xếp các dữ liệu để các chu kỳ E/P (Erase/Program) được phân đều giữa các block. Chu kì này được ghi lại và đếm vào **Raw value**. **Value** được tính theo đơn vị %, nghĩa là nó được bắt đầu từ 100 và giảm dần về 0 mỗi khi ổ đĩa được ghi vào. |
| 174 | **Unexpected power loss count** | | **Raw value** ghi lại số lần ổ cứng bị tắt đột ngột (như cúp điện, rút dây nguồn, ...), tích luỹ suốt quá lifetime của SSD.<br /> **Value**: luôn 100. |
| 175 | **Power Loss Protection Failure** | | (Updating ...) |
| 176 | **Erase Fail Count** | | **Raw value** cho biết số lần flash xoá lệnh thất bại. |
| 177 | **Wear Range Delta** | | |
| 179 | **Used Reserved Block Count Total** | | |
| 180 | **Unused Reserved Block Count Total** | | |
| 181 | **Program Fail Count Total**<br /><br />**Non-4K Aligned Access Count** | | **Raw value:** cho biết tổng số lần program flash không thành công.<br /> **Value:** Giảm dần từ 100 trở về 0, cho biết % số program không thành công còn lại được nhà sản xuất cho phép. |
| 182 | **Erase Fail Count** | | **Raw value:** cho biết tổng số lần xoá flash không thành công.<br /> **Value:** Giảm dần từ 100 trở về 0, cho biết % số  lần xoá flash không thành công còn lại được nhà sản xuất cho phép. |
| 183 | **SATA Downshift Error Count**<br /><br />**Runtime Bad Block**| | (WD, Samsung, Seagate)<br />**Raw value:** Thống kê số lần SATA interface bị giảm tín hiệu do gặp lỗi. Tham số này càng giảm chứng tỏ ổ đĩa đang gặp vấn đề hoặc bị xuống cấp.<br />**Value:** luôn 100. |
| 184 | **End-to-End error / IOEDC** | ![critical](/img/critical.png) | **End-to-end data** là các dữ liệu được chuyển từ host system (ví dụ như RAM cache) đến SSD và ngược lại.<br />**Raw value:** Thống kê số lần LBA (Logical Block Address) tag (hay parity bit) không khớp trong quá trình truyền end-to-end data.<br />**Value:** luôn 100. |
| 185 | **Head Stability** | | |
| 186 | **Induced Op-Vibration Detection** | | |
| 187 | **Reported Uncorrectable Errors** | ![critical](/img/critical.png) | **Raw value:** Hiển thị số lỗi không thể khôi phục bằng ECC.<br />**Value:** luôn 100. |
| 188 | **Command Timeout** | ![critical](/img/critical.png) | |
| 189 | **High Fly Writes** | | |
| 190 | **Temperature Difference**<br /><br />**Airflow Temperature** | | |
| 191 | **G-sense Error Rate** | | |
| 192 | **Power-off Retract Count** <br /> **Emergency Retract Cycle Count** <br /> **Unsafe Shutdown Count** | | |
| 193 | **Load Cycle Count** <br /> **Load/Unload Cycle Count** | | |
| 194 | **Temperature** <br /> **Temperature Celsius** | | |
| 195 | **Hardware ECC Recovered** | | |
| 196 | *Reallocation Event Count** | ![critical](/img/critical.png) | |
| 197 | **Current Pending Sector Count** | ![critical](/img/critical.png) | |
| 198 | **(Offline) Uncorrectable Sector Count[** | ![critical](/img/critical.png) | |
| 199 | **UltraDMA CRC Error Count** | | |
| 200 | **Multi-Zone Error Rate** | | |
| 201 | **Soft Read Error Rate**<br /><br />**TA Counter Detected** | ![critical](/img/critical.png) | |
| 202 | **Data Address Mark errors**<br /><br />**TA Counter Increased** | | |
| 203 | **Run Out Cancel** | | |
| 204 | **Soft ECC Correction** | | |
| 205 | **Thermal Asperity Rate** | | |
| 206 | **Flying Height** | | |
| 207 | **Spin High Current** | | |
| 208 | **Spin Buzz** | | |
| 209 | **Offline Seek Performance** | | |
| 210 | **Vibration During Write** | | |
| 211 | **Vibration During Write** | | |
| 212 | **Shock During Write** | | |
| 220 | **Disk Shift** | | |
| 221 | **G-Sense Error Rate** | | |
| 222 | **Loaded Hours** | | |
| 223 | **Load/Unload Retry Count** | | |
| 224 | **Load Friction** | | |
| 225 | **Load/Unload Cycle Count** | | |
| 226 | **Load 'In'-time** | | |
| 227 | **Torque Amplification Count** | | |
| 228 | **Power-Off Retract Cycle** | | |
| 230 | **GMR Head Amplitude(HDD)**<br /><br />**Drive Life Protection Status(SSD)** | | |
| 231 | **Life Left(SSD)**<br /><br />**Temperature** | | |
| 232 | **Endurance Remaining**<br /><br />**Available Reserved Space** | | |
| 233 | **Media Wearout Indicator(SSD)**<br /><br /> **Power-On Hours** | | |
| 234 | **Average erase count AND Maximum Erase Count** | | |
| 235 | **Good Block Count AND System(Free) Block Count** | | |
| 240 | **Head Flying Hours**<br /><br />**Transfer Error Rate** | | |
| 241 | **Total LBAs Written** | | |
| 242 | **Total LBAs Read** | | |
| 243 | **Total LBAs Written Expanded** | | |
| 244 | **Total LBAs Read Expanded** | | |
| 249 | **NAND Writes (1GiB)** | | |
| 250 | **Read Error Retry Rate** | | |
| 251 | **Minimum Spares Remaining** | | |
| 252 | **Newly Added Bad Flash Block** | | |
| 254 | **Free Fall Protection** | | |
