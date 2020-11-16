---
author: TraiOi
title:  "IPv4 Header Checksum"
date:   2020-11-15
categories: network
---
# 1. IPv4 header

## 1.1. OSI Model

**O**pen **S**ystems **I**nterconnection (**OSI**) model là một mô hình dựa theo thiết kế phân tầng, lý giải một cách trừu tượng kỹ thuật kết nối giữa các máy vi tính.

Mô hình **OSI** gồm 7 tầng (layer), mỗi tầng đều chỉ sử dụng chức năng của tầng bên dưới của nó, đồng thời chỉ cho phép tầng trên sử dụng các chức năng của mình. Các tầng này liên kết với nhau tạo thành một *protocol stack*.

![OSI model](/img/2020/11-15_01.png)

Trong đó, IP Protocol hay cụ thể hơn là **IPv4** hoạt động ở tầng 4 (Network Layer).

## 1.2. IPv4 header

**I**nternet **P**rotocol **v**ersion **4** (**IPv4**) là phiên bản thứ 4 của Internet Protocol (IP).

**IPv4 Header** bao gồm 14 trường (fields), trong đó 13 trường là bắt buộc, trường còn lại là trường **Options**.

![IPv4 Header](/img/2020/11-15_02.png)

### Example

Ví dụ có 1 IP header như sau:

```
4510 003c 16a7 4000 4006 a2b1 c0a8 0002 c0a8 0001
```

Trong đó:

| Value | Field | Bits | Explain |
| --- | --- | --- | --- |
| **4** | Version| [4] | IP version **4** |
| **5** | IHL | [4] | Độ dài của header là `5*(32/8)=20` bytes |
| **10** | TOS | [8] | Sử dụng để phân biệt gói tin trong QoS |
| **003c** | Total Length |[16] | Size of packet là `60` bytes |
| **16a7** | Identification | [16] | |
| **40** |  IP Flags | [3] | Không phân mảnh packet |
| **00** | Fragment Offset | [13] | |
| **40** | TTL | [8] | Time to live là `64` |
| **06** | Protocol | [8] | TCP packet |
| **a2b1** | **Header Checksum** | **[16]** | **Sử dụng cho kiểm tra lỗi** |
| **c0a8 0002** | Source Address | [32] | Source IP `192.168.0.2` |
| **c0a8 0001** | Destination Address | [32] | Dest IP `192.168.0.1` |

# 2. Checksum

## 2.1. OSI model checksum

**Checksum** là 1 đoạn binary/hex được sử dụng để kiểm tra sự toàn vẹn của gói tin. Phía **Sender** sẽ gửi gói tin và tính toán ra chuỗi **checksum**, sau đó chèn vào header. Phía **Receiver** sẽ nhận được gói tin và tính toán lại lần nữa. Sau đó, sử dụng chuỗi **checksum** đó để crosscheck với chuỗi **checksum** trong header mà phía **Sender** gửi đi, nếu trùng khớp thì gói tin là hợp lệ.

Trong mô hình OSI, mỗi tầng đều có 1 phương thức để kiểm tra lỗi khác nhau.
* **Layer 2:** Checksum sử dụng thuật toán CRC để kiểm tra lỗi của gói tin trước khi đi vào mô hình OSI.
* **Layer 3:** Các checksum tùy theo header của giao thức, ví dụ như **IPv4 header checksum**, ICMP header checksum, ..
* **Layer 4:** TCP pseduo header, reversed header, ..

## 2.2. IPv4 header checksum

**IPv4 Header Checksum** gồm 16 bits dùng cho việc kiểm tra lỗi của IPv4 header.

Thuật toán checksum của **IPv4 Header** được mô tả trong [RFC 791 - Internet Protocol](https://tools.ietf.org/html/rfc791#page-14).

> The checksum field is the 16 bit one's complement of the one's complement sum of all 16 bit words in the header. For purposes of computing the checksum, the value of the checksum field is zero.
>
> This is a simple to compute checksum and experimental evidence indicates it is adequate, but it is provisional and may be replaced by a CRC procedure, depending on further experience.

## 2.3. Calculate and Verify the checksum

### 2.3.1. Calculate the checksum

#### Bước 1: Convert các giá trị hexdump sang binary code

Do trường **Checksum** sử dụng 16 bit tương ứng với 4 giá trị hex nên sẽ chia header thành nhiều phần, mỗi phần 4 hex.

```
4510 -> 0100010100010000
003c -> 0000000000111100
16a7 -> 0001011010100111
4000 -> 0100000000000000
4006 -> 0100000000000110
0000 -> 0000000000000000 // Trường checksum được set bằng 0 để chèn vào sau
c0a8 -> 1100000010101000
0002 -> 0000000000000010
c0a8 -> 1100000010101000
0001 -> 0000000000000001
```

#### Bước 2: Cộng các giá trị binary lại với nhau.

```
4510 -> 0100010100010000
+
003c -> 0000000000111100
-------------------------
454c -> 0100010101001100 // sum
+
16a7 -> 0001011010100111
-------------------------
5bf3 -> 0101101111110011 // sum = sum + 0x16a7
+
4000 -> 0100000000000000
-------------------------
9bf3 -> 01001101111110011 // sum = sum + 0x4000
+
4006 -> 0100000000000110
-------------------------
dbf9 -> 01101101111111001 // sum = sum + 0x4006
+
0000 -> 0000000000000000
-------------------------
dbf9 -> 01101101111111001 // sum = sum + 0x0000
+
c0a8 -> 1100000010101000
-------------------------
19ca1 -> 011001110010100001 // sum = sum + 0xc0a8
9ca2 -> 1001110010100010 // sum = (sum & 0xffff) + (sum >> 16)
+
0002 -> 0000000000000010
-------------------------
9ca4 -> 01001110010100100 // sum = sum + 0x0002
+
c0a8 -> 1100000010101000
-------------------------
15d4c -> 010101110101001100 // sum = sum + 0xc0a8
5d4d -> 0101110101001101 // sum = (sum & 0xffff) + (sum >> 16)
+
0001 -> 0000000000000001
-------------------------
5d4e -> 0101110101001110 // sum = sum + 0001, result
```

#### Bước 3: NOT cho tổng các binary để lấy checksum

```
5d4e -> 0101110101001110
a2b1 -> 1010001010110001 // checksum
```

Sau đó **checksum** này sẽ được chèn vào header.

### 2.3.2. Verify the checksum

Khi **Receiver** nhận được gói tin sẽ tính toán tương tự như **Bước 2** ở mục **2.3.1**, tuy nhiên trường **Checksum** lúc này sẽ không còn là 0 nữa, mà sẽ là giá trị **Checksum** do **Sender** tính toán.

Gói tin được xét là hợp lệ nếu tổng từng giá trị 16 bits trong header bằng `0xffff`.

```
4510 + 003c + 16a7 + 4000 + 4006 + a2b1 + c0a8 + 0002 + c0a8 + 0001 = 2fffd // sum
(2fffd & 0xffff) + (2fffd >> 16) = fffd + 2 = ffff
```

**Receiver** xác sẽ xác định gói tin có  `~(0xffff)=0x0000` lỗi.

# 3. References

* [OSI Model (Wikipedia)](https://en.wikipedia.org/wiki/OSI_model)
* [IPv4 (Wikipedia)](https://en.wikipedia.org/wiki/IPv4)
* [IPv4 header checksum](https://en.wikipedia.org/wiki/IPv4_header_checksum)
