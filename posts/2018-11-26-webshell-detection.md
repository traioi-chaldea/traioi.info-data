---
author: TraiOi
title:  "WebShell Detection"
date:   2018-11-26
categories: 
  - malware
---
Hiểu được cách hoạt động, phân loại và cách phát hiện webshell. <!--more-->

# I. Giới thiệu về WebShell
## 1. Nguyên nhân xuất hiện WebShell

- Lỗ hổng bảo mật của Web Application.  
- Cấu hình sơ sài, không được filter chặt chẽ hoặc các cấu hình sai của Sysadmin trên Web Server hoặc Web Application cho các form input hoặc form upload file lên hệ thống.  
- Lỗ hổng bảo mật của hàm upload file.   
- Tài khoản user có quyền upload hoặc admin bị tổn hại (bị lộ password, bị khai thác, đăng nhập trên máy tính khác nhưng không logout, ..) hoặc những user không tin cậy nhưng được giao quyền upload.  
- Sử dụng các mã nguồn, plugin web không chính thống.  

## 2. WebShell là gì?

- WebShell là 1 dạng của RAT (Remote Access Trojan) hoặc backdoor. WebShell giúp hacker có thể tương tác được với Web Server qua các dòng lệnh.  
- WebShell có thể là 1 script hoặc 1 đoạn mã độc hại được chèn vào bên trong mã nguồn của web.  
- WebShell có thể bypass được các firewall layer 4 trở xuống vì hoạt động ở port 80 và 443 (Firewall thông thường sẽ allow web traffic qua 2 port này). Đối với các firewall layer 7 thì một số loại WebShell điển hình sẽ bị chặn dựa trên các chữ ký trong các rules tương ứng trên firewall.  
- Khi WebShell đã được upload lên Web Server thì hacker có thể tương tác với server để tiếp cận với các dịch vụ khác đang hoạt động trên cùng 1 server hoặc cùng vùng mạng thông qua các lỗ hổng bảo mật.  
- WebShell có thể được viết bằng PHP, ASP (Active Server Pages), JSP (JavaServer Pages) hoặc CGI (Common Gatewway Interface) là những ngôn ngữ server-side tương ứng với Web Server để dễ dàng thực thi các hàm của ngôn ngữ đó. Ngoài ra, WebShell cũng có thể chạy dưới các ngôn ngữ khác như Python, Perl, Bash Shell hoặc C nếu hacker có thể thay đổi được các file cấu hình Web Server như `htaccess`, `php.ini`, .. để cho phép thực thi các ngôn ngữ mà hacker mong muốn.  

## 3. Các tính năng nổi bật của WebShell

- Sử dụng làm backdoor trên hệ thống.  
- Thực thi các lệnh, thao tác database, .. với quyền tương ứng.    
- Sử dụng làm botnet để Ddos, spam mail, ...  
- Đào tiền ảo.  
- Thay đổi, sửa, xoá, upload, download, deface, .. trang web.  
- Local attack.  
- Một số WebShell có thể có khả năng leo thang đặc quyền trong hệ thống.  

## 4. Đặc điểm nhận dạng WebShell

- Các hàm đặc biệt có thể khai thác và sử dụng lệnh trực tiếp lên server.
  - Command Excecution:
    ```
    system
    passthru
    exec
    popen
    backticks (` `)
    pcntl_exec
    ```
  - PHP Code Execution:
    ```
    eval
    preg_replace('/.*/e, ..') (/e giống với eval())
    create_function
    assert
    include
    include_once
    require
    require_once
    ```  
- WebShell được obfuscation (làm rối mã), cách làm này sẽ khiến cho các Sysadmin khó khăn hơn trong việc phát hiện và phân tích WebShell. Các hàm thông dụng để obfuscate trong PHP:
  ```
  eval
  base64_decode
  gzinflate
  rote13
  preg_replace('/.*/e, ..') (/e giống với eval())
  ```
  - Ngoài ra WebShell còn có thể được làm rối mã bằng cách sử dụng các đoạn mã hex. Ví dụ:
  ```
  <?php ${"\x47L\x4f\x42\x41LS"}["\x6c\x68\x73l\x61wk"]="c";$kvbemdpsn="c";${"\x47\x4c\x4f\x42\x41\x4cS"}...
  ```

# II. Các cách phát hiện WebShell

## 1. Kiểm tra các files bị thay đổi gần nhất
```
find ./ -type file -mtime -7
```
Sau khi được thông báo là có xuất hiện những files lạ trên hệ thống, thì việc đầu tiên là mình sẽ tìm kiếm các files bị thay đổi gần nhất. Lúc này, mình sẽ khoanh vùng được số lượng và tên WebShell được upload lên (nếu có) trong khoảng thời gian đó. Thông thường, bước này sẽ được lặp lại nhiều lần để khoanh vùng, từ 3 ngày gần nhất đến 7 ngày, 14 ngày, tuỳ theo thời gian mà hệ thống thông báo hoặc nghi ngờ thời gian xuất hiện của WebShell.

## 2. Parse log
### 2.1. Khi chưa xác định được tên WebShell
- Ở phương pháp này, mình sẽ tận dụng kinh nghiệm trong xử lý WebShell để detect được tên WebShell trong log của Web Server dựa trên những sự xuất hiện khác biệt trong log, ví dụ như:  
  - 1/ Tên file kì lạ `92f66d.php`, `image.php.jpg`.  
  - 2/ Tần suất xuất hiện của file không hợp lý, như xuất hiện liên tục trong khoảng thời gian dài.  
  - 3/ Method `GET` xuất hiện những pattern đặc biệt liên quan tới hàm/lệnh tương tác với hệ thống như `system`, `ls`, `netstat`, ..  
  - 4/ Thông thường khi submit 1 form thông qua method `POST` đều phải đi qua một attr `action` trong form, bước này sẽ để lại `Referer` bên trong HTTP header. Nếu trong log xuất hiện method `POST` nhưng không đi kèm header `Referer` thì mình sẽ nghi ngờ file này là 1 WebShell hoặc hành vi khai thác lỗ hổng bảo mật.  
  - 5/ Method `GET` xuất hiện đường dẫn lạ, có thể là WebShell được include thông qua lỗ hổng RFI (Remote File Include).  

### 2.2. Khi đã xác định được tên WebShell
-  Khi đã xác định được tên của WebShell nhờ các phương pháp khác, mình sẽ `grep` trong log thời điểm mà WebShell được upload lên lần đầu tiên. Phương pháp này sẽ giúp tìm được cách thức mà WebShell được upload lên, hay lỗ hổng bảo mật mà WebShell được tận dụng để upload lên Web Server mà ngăn chặn triệt để.  

-  Ngoài ra, phương pháp này sẽ giúp mình trace ra được những WebShell khác được upload cùng thời điểm, hay những WebShell đã bị xoá, mình sẽ dựa vào tên của WebShell để cấu hình notify cho hệ thống. Sau này, hệ thống sẽ lập tức báo khi WebShell được upload lên.

## 3. Show processes
### 3.1. Network processes
```
netstat
ss
netcat
```
- WebShell được sử dụng làm backdoor có thể sử dụng để ẩn giấu trong traffic thông qua port web (80,443) hoặc được giấu ở những folder không nằm trong docroot của web như các folder `/home/`, `/tmp`, .. Các WebShell này sẽ không bị phát hiện khi Sysadmin thực hiện các công cụ Scan shell, hoặc các phương pháp detect WebShell thông thường. Nhưng các WebShell này cần phải mở 1 port khác port web để hacker có thể connect đến. Vì vậy, ở phương pháp này, mình sử dụng các lệnh để show các port của các dịch vụ đang hoạt động trên hệ thống, và kiểm tra các port lạ đang hoạt động để trace ra proccess của WebShell.  

- Ngoài các loại WebShell dùng làm backdoor này, các loại WebShell có các tính năng khác như sử dụng làm botnet, spam mail, .. cũng cần mở 1 socket đến C&C hoặc các máy victim, cũng có thể được detect bằng phương pháp này.

```
ngrep "^GET|^POST" -Wbyline |grep -v "########"
tcpdump -A -vv -i eth0 'port 80'
tshark -i eth0 -Y http.request.method==POST -Tfields -e text
```
- Đối với các WebShell sử dụng method `POST` được "ẩn mình" trong traffic của web, việc parse log rất khó khăn nên mình sẽ thực hiện monitoring bằng các công cụ theo dõi traffic, và xem các content trong traffic, nếu các content bất thường thì đó có khả năng là WebShell.

### 3.2. Program processes
 ```
top -c
ps
lsof
```
- Đối với các WebShell có các hoạt động bất thường như Ddos, spam mail, đào coin, .. thì sẽ cần sử dụng 1 lượng lớn tài nguyên của hệ thống. Dựa vào đặc điểm này, mình sẽ sử dụng các lệnh kiểm tra process để theo dõi các process đang hoạt động bất thường trên hệ thống như làm tiêu tốn CPU, RAM, xuất hiện nhiều lần với cùng 1 tên file, chiếm phần lớn hoạt động của các Web Server handler như `php-cgi`, `php`, `jsp`, ...

## 4. Dump database
- Một số mã nguồn web không được lưu dưới dạng file mà được lưu trong database sẽ khó xác định WebShell bằng các phương pháp dò quét mã nguồn thông thường. Vì vậy, ở các mã nguồn web dạng này phải dump database ra rồi sử dụng các phương pháp tìm kiếm chuỗi như `grep`, `egrep`, `awk`, .. để tìm kiếm qua các pattern của WebShell, từ đó trace được đoạn mã độc của WebShell.

## 5. Scan shell dựa trên mã nguồn
### 5.1. Dựa trên chữ ký
- Phương pháp này dựa trên các chữ ký hay các pattern của WebShell, các pattern này dựa trên một số đặc điểm đặc biệt của WebShell như:
  - Tên thường gặp của WebShell như `c99`, `r57`, `ApxSpy`, `WSO`, ..
  - Các hàm/lệnh mà các WebShell thường dùng `system`, `ls -l`, `cat /etc/passwd`, ..
  - Các loại ofuscation như hex, base64, ..
  - Các đặc điểm nhận dạng khác như file chỉ có 1 dòng, 0 dòng (no EOL), ..
- Phương pháp này đã được dùng từ lâu đời nhưng có một số điểm yếu như:
  - Các WebShell mới có chữ ký không nằm trong list chữ ký hiện có sẽ bypass được.
  - Phải thường xuyên cập nhật chữ ký của các loại WebShell.
  - Tốc độ scan khá chậm do sử dụng nhiều pattern để xử lý.
  - Có tỉ lệ scan nhầm cao.

### 5.2. Dựa trên thuật toán
Do kiến thức mình hạn hẹp :(, nên mình sẽ giải thích theo thuật toán dựa trên code của công cụ scan shell của Neopapsis là [NeoPI](https://github.com/Neohapsis/NeoPI). Đối với mình, đây là 1 công cụ rất mạnh trong việc phân tích và detect các loại WebShell đã được mã hoá hoặc obfuscated mã nguồn. Có gì thiếu sót thì mong các bạn thông cảm và góp ý giúp nhé :(.  

#### *a. Longest word:*
- Thông thường, các đoạn mã bị obfuscated sẽ là đoạn chuỗi dài nhất trong file. Đối với những hàm encoding, như `base64`, sẽ chứa một chuỗi dài không có khoảng trắng. Các đoạn văn bản hoặc script thông thường có độ dài tương đối ngắn, nên việc xác định chuỗi dài nhất trong file sẽ giúp khoanh vùng được những đoạn code bị obfuscated.
```
longest = 0
longest_word = ""
words = re.split("[\s,\n,\r]", data)
  if words:
    for word in words:
      length = len(word)
      if length > longest:
        longest = length
        longest_word = word
```
- **<u>Bước 1:</u>** NeoPI sẽ tách `data` thành các elements trong list `words` bằng các kí tự khoảng trắng, để đảm bảo các elements này sẽ là các chuỗi gồm các kí tự không có khoảng trắng.  
- **<u>Bước 2:</u>** Lần lượt lấy độ dài từng element trong list (`len(word)`) và so sánh với độ dài của element trước đó, nếu độ dài lớn hơn thì sẽ gán chuỗi đó vào biến `longest_word`. Khi kết thúc vòng lặp sẽ lấy ra được chuỗi dài nhất trong file.  

#### *b.Entropy:*
- Thông thường, các đoạn mã bị obfuscated sẽ có mật độ xuất hiện ngẫu nhiên của mỗi kí tự trong `data` là lớn hơn đoạn `data` bao gồm các kí tự có ý nghĩa. Entropy là 1 phương pháp phù hợp để tìm ra `data` bị mã hoá hoặc obfuscated dựa vào tính toán tần suất xuất hiện "hỗn loạn" của các kí tự trong `data`.  
```
entropy = 0
self.stripped_data =data.replace(' ', '')
for x in range(256):
	p_x = float(self.stripped_data.count(chr(x)))/len(self.stripped_data)
	if p_x > 0:
		entropy += - p_x * math.log(p_x, 2)
```
- **<u>Bước 1:</u>** NeoPI sẽ lượt bỏ các kí tự khoảng trắng để đảm bảo `data` chỉ bao gồm các kí tự.  
- **<u>Bước 2:</u>** NeoPI sẽ tính xác suất xuất hiện của 1 kí tự trong `data`. 
  - Trong Entropy thông tin, chúng ta quan tâm đến các byte dữ liệu trong bảng mã ASCII (mỗi byte có 2<sup>8</sup>=256 giá trị có thể xảy ra), vì vậy thang đo Entropy sẽ nằm trong khoảng từ 0 đến 8.  
	- 0: số lần xuất hiện có thứ tự
	- 8: số lần xuất hiện ngẫu nhiên
  - NeoPI sẽ chạy 1 vòng lặp đi qua mỗi giá trị `x` có thể xảy ra của 1 byte dữ liệu `range(256)`, và convert những giá trị (ở hệ 10) này sang dạng text `chr(x)` để thực hiện được phép tính đếm (`count`) trong `data`.  
  - Xác suất của kí tự `x` sẽ được tính bằng công thức:  
  - ![img](/img/webshell-xac-suat.png)  
  - Trong đó:   
    - `x` = `chr(x)`.  
    - `P(x)` là xác suất xuất hiện của kí tự `x`.  
	- `n(x)` là số lần xuất hiện của kí tự `x` trong `data`.  
	- `n(data)` là tổng số kí tự trong file.   
- **<u>Bước 3:</u>** Nếu kí tự `x` có xuất hiện trong `data` thì NeoPI sẽ tiếp tục tính toán Entropy nhị phân của các kí tự trong `data` bằng công thức:
  - ![img](/img/webshell-entropy.png) 
  - Trong đó:
    - `K` là 1 hằng số dựa trên đơn vị đo, ở trường hợp này `K` sẽ là 1.  
	- `n` là số các giá trị có thể có, ở đây `n` sẽ là `256`.  
	- `i` là giá trị rời rạc thứ `i`, ở đây `i` sẽ phụ thuộc vào lần lặp thứ `x` trong `range(256)`.  
	- `p(i)` là xác suất xuất hiện của kí tự thứ `i` trong bảng ASCII, ở đây `p(i)` sẽ tương đương với `P(x)`.  
- Khi kết thúc vòng lặp, NeoPI sẽ lấy được giá trị Entropy của từng file và đưa ra list các file với Entropy giảm dần. Các Entropy càng lớn càng có khả năng là WebShell.  

#### *c. I.C (Index of Coincidence):*
- Thông thường, các đoạn mã//văn bản có ý nghĩa sẽ có I.C (tạm dịch là Chỉ số trùng hợp ngẫu nhiên) của kí tự thường lớn, có nghĩa là các kí tự phân bố đều nhau trong `data`. Vì vậy I.C của đoạn mã có ý nghĩa sẽ lớn hơn I.C của đoạn mã bị mã hoá hoặc obfuscated.  
```
char_count = 0
total_char_count = 0
for x in range(256):
	char = chr(x)
	charcount = data.count(char)
	char_count += charcount * (charcount - 1)
	total_char_count += charcount
ic = float(char_count)/(total_char_count * (total_char_count - 1))
```
- **<u>Bước 1:</u>** NeoPI sẽ chạy vòng lặp để lấy các kí tự có thể trong bảng mã ASCII (`range(256)`), và convert các kí tự này từ hệ 10 sang dạng text `char = chr(x)`.  
- **<u>Bước 2:</u>** NeoPI sẽ đếm số lần xuất hiện của kí tự `char` trong `data` và gán vào `charcount`.  
- **<u>Bước 3:</u>** NeoPI sẽ tính chỉ số trùng hợp của `data` bằng công thức:
  - ![img](/img/webshell-ic.png)  
  - Trong đó:
    - `IC` là chỉ số trùng hợp ngẫu nhiên của `data`.  
	- `c` là số kí tự trong bảng mã ASCII, ở đây `c` là `256`.  
	- `i` là giá trị thứ `i` trong vòng lặp, ở đây `i` tương ứng với lần lặp thứ `x`.  
	- <code>n<sub>i</sub></code> là số lần xuất hiện của kí tự `char`, ở đây <code>n<sub>i</sub></code> tương ứng với `charcount`.  
- Khi kết thúc vòng lặp, NeoPI sẽ lấy được giá trị I.C và đưa ra list các file với I.C tăng dần. Các I.C càng lớn càng có khản năng là WebShell.  

### 5.3. Dựa trên Machine Learning
- (chưa nghĩ ra =)) )
