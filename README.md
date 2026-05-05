## Lab 2: Tấn công cạn kiệt tài nguyên DHCP Server (DHCP Starvation Attack)

### 1. Cấu hình DHCP Server

Thiết lập địa chỉ IP tĩnh cho DHCP Server:
![Cấu hình IP tĩnh cho DHCP Server](image-5.png)

Tiếp theo, tiến hành cấu hình dịch vụ DHCP trên Server để cấp phát IP cho mạng:
![Cấu hình dịch vụ DHCP](image-6.png)

### 2. Hành vi của người dùng hợp lệ ban đầu

Trên máy người dùng (PC0), thực hiện yêu cầu cấp phát IP từ DHCP Server. Kết quả cho thấy PC0 nhận được IP thành công:
![PC0 nhận IP từ DHCP](image-7.png)

### 3. Thực hiện tấn công cạn kiệt tài nguyên DHCP (DHCP Starvation)

Sử dụng Laptop đóng vai trò Kẻ tấn công (Attacker), sử dụng công cụ tạo lưu lượng (traffic generator) để liên tục gửi các yêu cầu xin cấp phát IP với các địa chỉ MAC giả mạo (MAC Spoofing). Mục đích là làm cạn kiệt toàn bộ dải địa chỉ IP (pool) của DHCP Server.
![Attacker thực hiện tấn công DHCP Starvation](image-8.png)

Kiểm tra lại việc xin cấp IP trên máy người dùng PC0, ta nhận thấy DHCP Server không còn địa chỉ IP nào trống để cấp phát thêm:
![PC0 không thể nhận IP do pool đã đầy](image-9.png)
Qua đó, cuộc tấn công cạn kiệt tài nguyên DHCP đã diễn ra thành công, gây ra tình trạng từ chối dịch vụ (DoS) với người dùng hợp pháp.

### 4. Triển khai cấu hình phòng thủ (DHCP Snooping)

Để ngăn chặn tấn công này, ta cấu hình tính năng bảo mật trên Switch. Cụ thể:

- Thiết lập cổng kết nối với DHCP Server thực sự là cổng tin cậy (Trust Port).
- Thiết lập giới hạn tốc độ yêu cầu (Rate Limit) trên các cổng không tin cậy (Untrust Ports) còn lại để ngăn chặn việc gửi hàng loạt thông điệp DHCP Discover.
  ![Cấu hình phòng thủ trên Switch](image-10.png)

### 5. Kiểm tra và xác minh hệ thống

Trên PC0, thực hiện yêu cầu cấp phát IP lại và nhận thấy máy đã được cấp IP thành công từ DHCP Server:
![PC0 nhận IP thành công sau phòng thủ](image-11.png)

Tiếp tục sử dụng Attacker để thực hiện lại tấn công. Kết quả cho thấy máy PC0 vẫn có thể xin được địa chỉ IP bình thường vì gói tin rác từ Attacker đã bị Switch loại bỏ hoặc giới hạn. Qua đó khẳng định quá trình phòng thủ thành công hoàn toàn cho bài lab.

## Lab 4: Cấu hình Port Security

### 1. Mô hình mạng

Mô hình bài lab bao gồm Router, Switch và 2 máy tính (PC0, PC1).
![Mô hình mạng](image1-lab4.png)

### 2. Thiết lập địa chỉ IP

- **Thiết lập IP cho PC0:**
  ![Địa chỉ IP PC0](image2-lab4.png)
- **Thiết lập IP cho PC1:**
  ![Địa chỉ IP PC1](image3-lab4.png)
- **Địa chỉ IP của Router:** `192.168.1.1`

### 3. Cấu hình trên Router

Sau khi cài đặt địa chỉ IP và Subnet Mask cho Router, tiến hành bật các cổng kết nối đến Switch và Cloud:
![Cấu hình cổng trên Router](image4-lab4.png)

Kiểm tra trạng thái các cổng trên Router để đảm bảo cấu hình đã được áp dụng và cổng đã bật (Up):
![Kiểm tra trạng thái Port trên Router](image5-lab4.png)

### 4. Cấu hình Port Security trên Switch

Trên Switch, ta tiến hành cấu hình Port Security tại cổng kết nối với PC (giả sử cổng Fa0/1) với các thông số:

- **Maximum 1:** Chỉ cho phép duy nhất 1 thiết bị (1 địa chỉ MAC) truy cập thông qua cổng này.
- **Mac-address sticky:** Switch tự động học và ghi nhớ vĩnh viễn địa chỉ MAC của thiết bị đầu tiên kết nối vào cổng.
- **Violation shutdown:** Khi phát hiện thiết bị khác có địa chỉ MAC không khớp kết nối vào, cổng sẽ lập tức bị đóng chặn (Shutdown).

![Cấu hình Port Security trên Switch](image6-lab4.png)

### 5. Kiểm tra và xác minh hệ thống

**Bước 5.1: Ghi nhận địa chỉ MAC hợp lệ (PC0)**
Từ PC0, tiến hành gửi gói tin ping đến địa chỉ IP của Router (`192.168.1.1`). Mục đích là để kích hoạt luồng dữu liệu, giúp Switch học được địa chỉ MAC của PC0.
![PC0 ping Router thành công](image7-lab4.png)

Kiểm tra trên Switch, cấu hình đã lưu lại địa chỉ MAC của PC0 thông qua tính năng Sticky MAC:
![Switch lưu địa chỉ MAC PC0](image8-lab4.png)

**Bước 5.2: Kiểm tra chế độ tự động chặn thiết bị lạ (PC1)**
Giả lập tấn công hoặc thay đổi thiết bị trái phép bằng cách rút dây mạng từ PC0 và cắm sang PC1:
![Đổi kết nối cáp sang PC1](image9-lab4.png)

Từ PC1, thực hiện ping tới Router. Kết quả trả về cho thấy ping thất bại (Destination host unreachable / Request timed out), PC1 đã bị hệ thống chặn kết nối.
![PC1 ping Router thất bại](image10-lab4.png)

Trên CLI của Switch, ta nhận được log cảnh báo cổng đã bị chuyển sang trạng thái đóng lập tức (line protocol down).
![Switch cảnh báo đóng cổng](image11-lab4.png)

Sử dụng lệnh kiểm tra chi tiết tham số Port Security trên cổng đó (Ví dụ: `show port-security interface fa0/1`), ta thấy số lần vi phạm `Security Violation Count` tăng lên và trạng thái cổng (Port Status) biến thành `Secure-down`.
![Trạng thái chi tiết của chức năng Port Security](image12-lab4.png)

### 6. Kết luận

Bài lab đã triển khai thành công cơ chế **Port Security** kích hợp tính năng **Sticky MAC**. Giải pháp giúp Switch có khả năng tự động nhận diện và chỉ cấp quyền mạng cho thiết bị hợp pháp đầu tiên (PC0). Đồng thời, hệ thống chủ động ngăn chặn và phòng thủ hiệu quả trước mọi sự truy cập trái phép của thiết bị lạ (PC1) bằng cơ chế tự động khóa cổng (Shutdown) khi có thay đổi phần cứng vi phạm chính sách bảo mật mạng.

## Lab 5: Cấu hình Dynamic ARP Inspection DAI

Sơ đồ mạng:
![alt text](image-19.png)
Cấu hình trên Switch, đưa tất cả các cổng vào cùng một VLAN và cấu hình DHCP Snooping (vì DAI cần cơ sở dữ liệu này để hoạt động)
![alt text](image-12.png)
Trên Router DHCP, cấu hình IP cho DHCP và thực hiện việc tạo DHCP pool và khai báo mạng cấp phát
![alt text](image-13.png)
Trên Router Host, thực hiện việc xin IP từ DHCP
![alt text](image-14.png)
Trên Switch, kiểm tra bảng IP, MAC, Port, VLAN mà DHCP Snooping đã học
![alt text](image-15.png)
Tiếp theo, chúng ta sẽ bật tính năng bảo mật DAI trên VLAN 123. Theo mặc định, tất cả các giao diện sẽ là untrusted (không tin cậy). Tuy nhiên, cổng kết nối với Router DHCP cũng cần được đặt là Trusted cho ARP, nếu không các gói ARP từ Router gửi về cũng sẽ bị chặn
![alt text](image-16.png)
Trên Router Attack, giả sử kẻ tấn công cố gắng giả mạo IP của Host (192.168.1.1) để ping đến DHCP
![alt text](image-17.png)
Kết quả lệnh ping từ Attack:
![alt text](image-18.png)

Ping thất bại, điều này xảy ra vì Switch kiểm tra gói ARP từ cổng Fa0/2, thấy IP 192.168.1.1 nhưng MAC address lại không khớp với những gì nó đã lưu trong bảng DHCP Snooping (vốn ghi nhận IP đó thuộc về cổng Fa0/1)

## Lab 3: Tấn công giả mạo DHCP Server và Phòng chống bằng DHCP Snooping

### 1. Mô hình mạng
Mô hình bài lab bao gồm PC đóng vai trò là DHCP Client, một Switch trung tâm (L2 Switch), một DHCP Server hợp lệ (Original Server) và một DHCP Server giả mạo (Fake Server).
![Mô hình mạng](image-20.png)

### 2. Cấu hình DHCP Server hợp pháp và giả mạo

- **Cấu hình IP và dịch vụ DHCP trên Original Server:**
  ![Cấu hình IP Original Server](image-21.png)
  ![Cấu hình dịch vụ DHCP Original Server](image-22.png)

- **Cấu hình IP và dịch vụ DHCP trên Fake Server:**
  ![Cấu hình IP Fake Server](image-23.png)
  ![Cấu hình dịch vụ DHCP Fake Server](image-24.png)

### 3. Cấu hình phòng thủ DHCP Snooping trên Switch

Để ngăn chặn DHCP Server giả mạo cấp phát IP sai lệch cho người dùng, ta tiến hành cấu hình DHCP Snooping trên Switch để phân biệt các nguồn cấp phát đáng tin cậy.

- **Bật tính năng DHCP Snooping toàn cục:**
  ![Bật DHCP Snooping toàn cục trên Switch](image-25.png)

- **Thiết lập cổng tin cậy (Trusted Port):**
  Cấu hình cổng kết nối với Original Server thành cổng tin cậy để cho phép các thông điệp cấp phát DHCP (`DHCP OFFER`, `DHCP ACK`) đi qua. Các cổng kết nối tới PC và Fake Server theo mặc định sẽ là cổng không tin cậy (Untrusted Port), tức là sẽ chặn các thông điệp phản hồi DHCP từ Server.
  ![Thiết lập cổng tin cậy trên Switch](image-26.png)

### 4. Kiểm tra và xác minh hệ thống

- **Kiểm tra cấp phát IP trên máy Client (PC0):**
  Trên PC0, thực hiện yêu cầu xin cấp phát lại địa chỉ IP thông qua DHCP. Kết quả cho thấy PC0 nhận được địa chỉ IP tương ứng với dải mạng được cấp phát từ Original Server. Switch đã chặn thành công gói tin `DHCP OFFER` từ Fake Server (đang kết nối qua cổng không tin cậy như `Fa0/2`).
  ![PC0 nhận IP hợp lệ](image-27.png)

- **Kiểm tra trạng thái DHCP Snooping trên Switch:**
  Xem trạng thái và cơ chế hoạt động của DHCP Snooping trên các cổng:
  ![Kiểm tra trạng thái các cổng](image-28.png)

- **Xem bảng dữ liệu DHCP Snooping Binding:**
  Kiểm tra danh sách các máy khách đã được cấp phát IP hợp lệ và được ghi nhận với các thông số như địa chỉ MAC, IP, Lease time, VLAN, và Interface:
  ![Danh sách IP MAC hợp lệ](image-29.png)

### 5. Kết luận
Với tính năng **DHCP Snooping**, Switch đã giám sát quy trình xin - cấp IP và tự động làm rơi (drop) các thông điệp phản hồi từ máy chủ DHCP giả mạo. Từ đó giúp hệ thống mạng chỉ nhận được thông tin cấu hình IP từ máy chủ DHCP hợp lệ duy nhất, đảm bảo tính an toàn dữ liệu và phòng ngừa tấn công đánh chặn (Man-in-the-Middle) trong nội bộ mạng Lan.