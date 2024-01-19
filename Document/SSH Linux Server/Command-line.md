# Tạo và cài đặt máy ảo VPS Centos để triển khai ứng dụng thực tế
VPS Centos rất ổn định và phù hợp để triển khai ứng dụng thực tế.

## Phần 1: Tổng hợp các lênh trong Centos 7: 

### Quản lý gói

| Lệnh | Mô tả | Ví dụ | 
|-|-|-|
| `sudo yum install <package>` | Cài đặt gói | `sudo yum install httpd` |
| `sudo yum update` | Cập nhật các gói |  |
| `sudo yum remove <package>` | Gỡ cài đặt gói | `sudo yum remove httpd` |
| `sudo yum search <keyword>` | Tìm kiếm gói | `sudo yum search httpd` |
| `sudo yum clean all` | Xóa bộ nhớ cache của yum | |
| `sudo yum history` | Xem lịch sử cài đặt/update gói | |
| `sudo yum info <package>` | Xem thông tin về một gói | `sudo yum info httpd` |
| `rpm -qa` | Liệt kê tất cả các gói đã cài | | 
| `rpm -qi <package>` | Xem thông tin gói đã cài | `rpm -qi httpd` |
| `rpm -qf <file>` | Xác định gói sở hữu file | `rpm -qf /etc/httpd/conf/httpd.conf` |

### Quản lý dịch vụ 

| Lệnh | Mô tả | Ví dụ |
|-|-|-|
| `sudo systemctl start/stop/restart <service>` | Khởi động/dừng/khởi động lại dịch vụ | `sudo systemctl start httpd` |
| `sudo systemctl enable/disable <service>` | Bật/tắt khởi động cùng hệ thống cho dịch vụ | `sudo systemctl enable httpd` |
| `sudo journalctl -u <service>` | Xem log của dịch vụ | `sudo journalctl -u httpd` |

### Quản lý mạng

| Lệnh | Mô tả | Ví dụ | 
|-|-|-|
| `sudo firewall-cmd --add-port=<port>/tcp` | Mở port trên firewall | `sudo firewall-cmd --add-port=80/tcp` |
| `sudo nano /etc/sysconfig/network-scripts/ifcfg-<interface>` | Chỉnh sửa cấu hình mạng | `sudo nano /etc/sysconfig/network-scripts/ifcfg-ens33` |
| `sudo nmcli connection show` | Xem các kết nối mạng | |
| `sudo nmcli connection up/down <name>` | Bật/tắt kết nối mạng | `sudo nmcli connection up eth0` |
| `sudo firewall-cmd --list-all` | Xem các rules firewall | |
| `ping <ip>` | Kiểm tra kết nối mạng | `ping 192.168.1.1` |
| `traceroute <ip>` | Xem đường đi gói tin tới IP | `traceroute google.com` |
| `netstat -antup` | Kiểm tra các kết nối mạng | |  
| `ifconfig` | Xem thông tin cấu hình mạng | |
| `iwconfig` | Xem thông tin cấu hình wifi | |

### Quản lý tài nguyên

| Lệnh | Mô tả | Ví dụ |
|-|-|-| 
| `top` | Hiển thị các tiến trình đang chạy | |
| `ps aux` | Hiển thị các tiến trình chi tiết | | 
| `kill <pid>` | Kết thúc một tiến trình | `kill 1234` |
| `df -h` | Kiểm tra dung lượng ổ đĩa | |
| `du -sh <directory>` | Kiểm tra dung lượng thư mục | `du -sh /var/log` |

### Quản lý user

| Lệnh | Mô tả | Ví dụ |
|-|-|-|
| `sudo useradd <username>` | Tạo người dùng mới | `sudo useradd john` |  
| `sudo passwd <username>` | Đổi mật khẩu người dùng | `sudo passwd john` |
| `sudo visudo` | Chỉnh sửa quyền sudo | |

### Lệnh file

| Lệnh | Mô tả | Ví dụ |
|-|-|-|
| `chmod` | Thay đổi quyền truy cập file | `chmod +x file.sh` |
| `chown` | Thay đổi quyền sở hữu file | `chown user:group file.txt` |

### Lệnh mạng

| Lệnh | Mô tả | Ví dụ |
|-|-|-| 
| `scp` | Sao chép file qua SSH | | 
| `scp <file> <username>@<ip>:<path>` | Sao chép file tới máy từ xa | `scp file.txt john@192.168.1.100:/home/john` |
| `scp <username>@<ip>:<file> <local_path>` | Sao chép file từ máy từ xa | `scp john@192.168.1.100:/home/john/file.txt .` |
| `rsync` | Đồng bộ hóa file/thư mục | |
| `rsync -r <source> <destination>` | Đồng bộ thư mục | `rsync -r /home/john/folder /backup` | 
| `rsync -a <source> <username>@<ip>:<dest>` | Đồng bộ từ xa | `rsync -a /home/john john@192.168.1.100:/backup` |
| `rsync -az <ip>:<src> <dest>` | Đồng bộ từ xa | `rsync -az 192.168.1.100:/home/john/folder /backup` |

### Lệnh sFTP

| Lệnh | Mô tả | Ví dụ |
|-|-|-|
| `sftp <username>@<ip>` | Kết nối tới máy chủ sFTP | `sftp john@192.168.1.100` |
| `ls` | Liệt kê file và thư mục | |  
| `pwd` | Hiển thị thư mục làm việc hiện tại | |
| `cd <path>` | Chuyển đến thư mục | `cd /home/john` |
| `mkdir <dir>` | Tạo thư mục mới | `mkdir folder1` |  
| `rm <file>` | Xóa file | `rm file.txt` |
| `rmdir <dir>` | Xóa thư mục rỗng | `rmdir empty_folder` |
| `get <file>` | Tải file xuống máy local | `get file.txt` |
| `put <file>` | Tải file lên máy chủ | `put local_file.txt` |
| `exit` | Thoát khỏi kết nối sFTP | |


### Đọc nội dung file 

| Lệnh | Mô tả | Ví dụ |
|-|-|-|  
| `cat <file>` | In nội dung file ra màn hình | `cat /etc/passwd` |
| `less <file>` | Xem nội dung file dạng trang | `less /var/log/messages` |
| `head <file>` | Hiển thị 10 dòng đầu của file | `head /etc/passwd` |
| `tail <file>` | Hiển thị 10 dòng cuối của file | `tail /var/log/messages` |
| `tail -f <file>` | Theo dõi nội dung file thời gian thực | `tail -f /var/log/syslog` |  
| `grep <pattern> <file>` | Tìm kiếm theo mẫu trong file | `grep "error" /var/log/messages` |

## Phần 2: tổng hợp các cú pháp Markdown cho GitHub README.md: 

1. Tiêu đề

# Tiêu đề cấp 1
## Tiêu đề cấp 2
### Tiêu đề cấp 3
#### Tiêu đề cấp 4  
##### Tiêu đề cấp 5
###### Tiêu đề cấp 6
2. Định dạng chữ

**In đậm**
*In nghiêng*  
~~Gạch ngang~~
3. Danh sách

- Item 1
- Item 2
  - Item 2.1
  - Item 2.2
4. Link

[Text hiển thị](https://www.example.com)
5. Hình ảnh

![Alt text](image.jpg)
6. Bảng

| Cột 1 | Cột 2 | Cột 3 |
|-|-|-|  
| Hàng 1 | Ô 2 | Ô 3|
| Hàng 2 | Ô 5 | Ô 6|
7. Block code

```
// Code block
```
8. Gạch đầu dòng

- [ ] Task 1
- [x] Task 2
9. Thanh ngăn cách

---
10. Ký tự đặc biệt

\! \# \$ \% \&
11. Nhấn mạnh

==Nội dung cần nhấn mạnh==
12. Chú thích

Hamlet đã nói:

> To be, or not to be, that is the question
13. Emoji

:smile:
14. table nâng cao

<table>
  <tr>
    <th>Company</th>
    <th>Contact</th>
    <th>Country</th>
  </tr>
  <tr>
    <td>Alfreds Futterkiste</td>
    <td>Maria Anders</td>
    <td>Germany</td>
  </tr>
  <tr>
    <td>Centro comercial Moctezuma</td>
    <td>Francisco Chang</td>
    <td>Mexico</td>
  </tr>
</table>
