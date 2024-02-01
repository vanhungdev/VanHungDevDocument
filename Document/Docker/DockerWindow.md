# Tải và cài đặt docker trên window

Trong phần này sẽ hướng dẫn tải và cài đặt docker trên window.

## Phần 1: Cài đặt WSL 2:

Có một số lý do chính tại sao Docker trên Windows cần WSL 2:

 - WSL 2 sử dụng một kiến trúc ảo hóa Linux hoàn chỉnh thay vì chỉ phần userspace như WSL 1. Điều này cho phép chạy máy ảo Linux đầy đủ hơn. Docker yêu cầu một môi trường Linux đầy đủ.  
 - WSL 2 sử dụng hypervisor Hyper-V của Windows để cung cấp các máy ảo Linux. Điều này cho phép tích hợp chặt chẽ và hiệu suất tốt hơn so với WSL 1.  
 - WSL 2 hỗ trợ chạy các ứng dụng Linux sử dụng các tính năng như systemd mà trước đây không khả dụng trên WSL 1. Docker sử dụng systemd và các tính năng này.  
 - WSL 2 cho phép truy cập vào các tính năng phần cứng như CPU và bộ nhớ một cách trực tiếp hơn. Điều này cải thiện hiệu năng Docker.  
 - Giao diện mạng trên WSL 2 (sử dụng vEthernet) cho phép Docker hoạt động tốt hơn, khắc phục một số vấn đề về mạng trên WSL 1.  
 - Như vậy WSL 2 cung cấp môi trường Linux tốt hơn, tích hợp chặt chẽ hơn với Windows, cho phép Docker hoạt động hiệu quả và ổn định hơn.

Các bước cài đặt:  
 ```bash
	Step 1 - Enable the Windows Subsystem for Linux
	Step 2 - Enable Virtual Machine feature
	Step 3 - Download the Linux kernel update package
	Step 4 - Set WSL 2 as your default version
 ```


**Step 1 - Enable the Windows Subsystem for Linux**

Enable chức năng Windows Subsystem for Linux  

 ```bash
  dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
 ```


**Step 2 - Enable Virtual Machine feature**  

Trước khi cài đặt WSL 2, bạn phải kích hoạt tính năng tùy chọn Virtual Machine Platform . Máy của bạn sẽ yêu cầu khả năng ảo hóa để sử dụng tính năng này.  

 ```bash
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
 ```

Khởi động lại máy tính  

**Step 3 - Download the Linux kernel update package**

Mở PowerShell với quyền Administrator (Start menu > PowerShell > right-click > Run as Administrator) Và chạ cácy lệnh sau:  

 ```bash

   wsl.exe --update

 ```

**Step 4 - Set WSL 2 as your default version**  

Mở PowerShell và chạy lệnh này để đặt WSL 2 làm phiên bản mặc định khi cài đặt bản phân phối Linux mới:  

 ```bash
wsl --set-default-version 2
 ```

Cần khởi động lại máy  


## Phần 2: Tải docker desktop:  

**Tải docker desktop tại trang chủ đường link sau**  

 ```bash
https://www.docker.com/products/docker-desktop/
 ```

Tải và cài đặt như bình thường

**Kiểm tra đã cài đặt thành công hay được chưa**    

Mở teminal chạy lệnh sau:  

 ```bash
 Docker ps
 ```
