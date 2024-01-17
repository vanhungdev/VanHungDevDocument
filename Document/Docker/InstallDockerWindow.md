# Tải và cài đặt docker trên window

Trong phần này sẽ hướng dẫn tải và cài đặt docker trên window.

## Phần 1: Cài đặt docker trên window:

	Step 1 - Download the Linux kernel update package
	Step 2 - Enable the Windows Subsystem for Linux
	Step 3 - Enable Virtual Machine feature
	Step 4 - Set WSL 2 as your default version


Tải docker desktop tại trang chủ đường link sau  

 ```bash
https://www.docker.com/products/docker-desktop/
 ```

**Step 1 - Download the Linux kernel update package**

Mở PowerShell với quyền Administrator (Start menu > PowerShell > right-click > Run as Administrator) Và chạ cácy lệnh sau:  

 ```bash

   wsl.exe --install
   wsl.exe --update

 ```
**Step 2 - Enable the Windows Subsystem for Linux**

Enable chức năng Windows Subsystem for Linux  

 ```bash
  dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
 ```


**Step 3 - Enable Virtual Machine feature**
Trước khi cài đặt WSL 2, bạn phải kích hoạt tính năng tùy chọn Virtual Machine Platform . Máy của bạn sẽ yêu cầu khả năng ảo hóa để sử dụng tính năng này.  

 ```bash
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
 ```
**Step 4 - Set WSL 2 as your default version**
Mở PowerShell và chạy lệnh này để đặt WSL 2 làm phiên bản mặc định khi cài đặt bản phân phối Linux mới:  

 ```bash
wsl --set-default-version 2
 ```
