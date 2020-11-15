---
author: TraiOi
title:  "Get SNMP Metrics with SNMP Exporter"
date:   2020-10-13
categories: monitoring
---
# 1. SNMP Architecture

## 1.1. What is SNMP

**SNMP** (Simple Network Management Protocol) là giao thức sử dụng để giám sát và điều khiển mạng như Switch, Router, Server, ..

## 1.2. How it work?

**SNMP** hoạt động dựa trên kiến trúc Client-Server với Client đóng vai trò như Manager và Server như một Agent.
* **Agent:** lưu trữ MIB Database (Management Information Base) chứa các thông tin về hoạt động của thiết bị mà agent đang giám sát.
* **Manager:** thu thập các thông tin mà Agent gửi qua giao thức **SNMP**.

![center](/img/2020/1310-01.png)

## 1.3. Understand SNMP MIB and OIDs

* **MIB** (Management Information Base) định nghĩa các thông tin về hoạt động của thiết bị được giám sát thông qua giao thức **SNMP** theo một cấu trúc phân cấp. Mỗi MIB sử dụng cấu trúc cây thư mục (được định nghĩa trong ISO ASN.1) để định nghĩa các thông tin sẵn có, mỗi mẫu thông tin trong cây là một nút có nhãn (Labeled node) được tạo thành bởi **OID** (Object Identifier) và mô tả thuộc tính của **OID** đó.

* **OID** (Object Identifier) là dãy số nguyên, định danh cho một nhánh, được tách ra bởi các dấu chấm.
  * Các **OID** đều được định nghĩa trong [OID ref](https://oidref.com/).
  * ***Ví dụ:***
    * Để lấy thông tin của *sysUpTime* từ MIB, ta có **OID** là `1.3.6.1.2.1.1.3`.
    * *Trong đó:* Nhánh của cây thư mục được biểu diễn như link [OID-1.3.6.1.2.1.1.3](https://oidref.com/1.3.6.1.2.1.1.3).

![center](/img/2020/1310-02.png)

## 1.4. SNMP Versions

* **SNMP v1:** Dùng phương thức xác thực đơn giản với community string.
* **SNMP v2:** Phát triển từ v1 nhưng thêm vào các thông điệp Getbulk và Inform (manager-to-manager).
* **SNMP v3:** Dùng phương thức xác thực cải tiến hơn về bảo mật như mã hóa (DES, AES) dữ liệu và hàm băm (MD5, SHA1).

## 1.5. SNMP Operations

* *Pull data from Manager:*
  * **GetRequest:** Gửi yêu cầu agent cấp thông tin dựa vào OID.
  * **SetRequest:** Đặt giá trị cho đối tượng của agent dựa vào OID.
  * **GetNextRequest:** Gửi yêu cầu agent cấp thông tin kế tiếp OID đó trong MIB.
  * **GetBulkRequest:** Gửi yêu cầu agent cấp nhiều thông tin dựa vào nhiều OIDs trong GetBulk.
* *Push data to SNMP Server:*
  * **Event traps:** Alert về bất kỳ thông tin event nào xảy ra trong Agent như lỗi giao diện, ngắt VPN, ...

# 2. SNMP Exporter

## 2.1. What is an SNMP Exporter

* **SNMP Exporter** là công cụ thu thập data từ agent và hiển thị chúng ở định dạng được sử dụng bởi Prometheus.
* **SNMP Exporter** là opensource được publish bởi Prometheus \[[Link](https://github.com/prometheus/snmp_exporter)\].

## 2.2. How it work
