# Tạo và cài đặt máy ảo VPS Centos để triển khai ứng dụng thực tế
VPS Centos rất ổn định và phù hợp để triển khai ứng dụng thực tế.

## Phần 1: Tổng hợp các lênh trong Centos 7: 

| COMMAND | DESCRIPTION |
|-|-|
| `sudo yum install <package>` | Cài đặt gói |  
| `sudo yum update` | Cập nhật các gói |
| `sudo yum remove <package>` | Gỡ cài đặt gói |
| `sudo yum search <keyword>` | Tìm kiếm gói |
| `sudo systemctl start/stop/restart <service>` | Khởi động/dừng/khởi động lại dịch vụ |  
| `sudo systemctl enable/disable <service>` | Bật/tắt khởi động cùng hệ thống cho dịch vụ |
| `sudo firewall-cmd --add-port=<port>/tcp` | Mở port trên firewall |
| `sudo nano /etc/sysconfig/network-scripts/ifcfg-<interface>` | Chỉnh sửa cấu hình mạng |
| `sudo nmcli connection show` | Xem các kết nối mạng |
| `sudo nmcli connection up/down <name>` | Bật/tắt kết nối mạng |
| `sudo useradd <username>` | Tạo người dùng mới |
| `sudo passwd <username>` | Đổi mật khẩu người dùng |  
| `sudo visudo` | Chỉnh sửa quyền sudo | 
| `sudo journalctl -u <service>` | Xem log của dịch vụ |
| `sudo firewall-cmd --list-all` | Xem các rules firewall |
| `sudo yum clean all` | Xóa bộ nhớ cache của yum |  
| `sudo yum history` | Xem lịch sử cài đặt/update gói |
| `sudo yum info <package>` | Xem thông tin về một gói |
| `rpm -qa` | Liệt kê tất cả các gói đã cài |
| `rpm -qi <package>` | Xem thông tin gói đã cài | 
| `rpm -qf <file>` | Xác định gói sở hữu file |
| `top` | Hiển thị các tiến trình đang chạy |
| `ps aux` | Hiển thị các tiến trình chi tiết |
| `kill <pid>` | Kết thúc một tiến trình |
| `chmod` | Thay đổi quyền truy cập file |
| `chown` | Thay đổi quyền sở hữu file |
| `df -h` | Kiểm tra dung lượng ổ đĩa |
| `du -sh <directory>` | Kiểm tra dung lượng thư mục |  
| `scp` | Sao chép file qua SSH |
| `scp <file> <username>@<ip>:<path>` | Sao chép file tới máy từ xa | 
| `scp <username>@<ip>:<file> <local_path>` | Sao chép file từ máy từ xa |
| `rsync` | Đồng bộ hóa file/thư mục |
| `rsync -r <source> <destination>` | Đồng bộ thư mục |
| `rsync -a <source> <username>@<ip>:<dest>` | Đồng bộ từ xa |  
| `rsync -az <ip>:<src> <dest>` | Đồng bộ từ xa |
| `ping <ip>` | Kiểm tra kết nối mạng | 
| `traceroute <ip>` | Xem đường đi gói tin tới IP |
| `netstat -antup` | Kiểm tra các kết nối mạng |
| `ifconfig` | Xem thông tin cấu hình mạng |
| `iwconfig` | Xem thông tin cấu hình wifi |
