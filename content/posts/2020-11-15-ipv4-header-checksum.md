---
author: TraiOi
title:  "IPv4 Header Checksum"
date:   2020-10-15
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
4500 0073 0000 4000 4006 b861 c0a8 0001 c0a8 00c7 
```

Trong đó:

* `4` - **Version**[4] - IP version **4**.
* `5` - **IHL**[4] - Độ dài của header là `5*(32/8)=20` bytes.
* `00` - **TOS**[8] - Sử dụng để phân biệt gói tin trong QoS.
* `0073` - **Total Length**[16] - Size of packet là `115` bytes.
* `0000` - **Identification**[16].
* `40` - **IP Flags**[3] - Không phân mảnh packet.
* `00` - **Fragment Offset**[13].
* `40` - **TTL**[8] - Time to live là `64`.
* `06` - **Protocol**[8] - TCP packet.
* `b861` - ***Header Checksum**[16]) - Sử dụng cho kiểm tra lỗi*.
* `c0a8 0001` - **Source Address**[32] - Source IP `192.168.0.1`.
* `c0a8 00c7` - **Destination Address**[32] - Dest IP `192.168.0.199`.

# 2. Checksum

## 2.1. OSI model checksum


## 1.3. Calculate and verify the checksum

Thuật toán checksum của **IPv4 Header** được mô tả trong [RFC 791 - Internet Protocol](https://tools.ietf.org/html/rfc791#page-14).

> The checksum field is the 16 bit one's complement of the one's complement sum of all 16 bit words in the header. For purposes of computing the checksum, the value of the checksum field is zero.
>
> This is a simple to compute checksum and experimental evidence indicates it is adequate, but it is provisional and may be replaced by a CRC procedure, depending on further experience.

### 1.3.1. Calculate the checksum



### 1.3.2. Verify the checksum

# 2. Demo with code

# 3. References

* [OSI Model (Wikipedia)](https://en.wikipedia.org/wiki/OSI_model)
* [IPv4 (Wikipedia)](https://en.wikipedia.org/wiki/IPv4)