
## Package

Để chạy được tự động sinh ra model chúng ta cần add thư viện sau  
Đảm bảo các package phải cùng version

```bash
<PackageReference Include="Microsoft.AspNetCore.Identity.EntityFrameworkCore" Version="5.0.12" />
<PackageReference Include="Microsoft.EntityFrameworkCore.Design" Version="5.0.12" />
<PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" Version="5.0.12" />
<PackageReference Include="Microsoft.EntityFrameworkCore.Tools" Version="5.0.12"/>
```

## Set as Setup Project

Đảm bảo project build được không vấn đề gì.

## Script chạy gen model

```bash
Scaffold-DbContext "Server=.\SQLExpress;Database=MedicalExaminationDB;User Id=sa;Password=123456;" Microsoft.EntityFrameworkCore.SqlServer -OutputDir Models

```

Chú ý chỗ Server= nếu `DESKTOP-HF7A88S\SQLEXPRESS` thì để `.\SQLEXPRESS`  
Nếu trên host thì để cả host lẫn port ví dụ `172.29.28.17,1433``


```bash
Scaffold-DbContext "Server=34.125.122.249,1433;Database=ParkingSystemDB;User Id=sa;Password=Provanhung77;" Microsoft.EntityFrameworkCore.SqlServer -OutputDir Models -Force
```

Chú ý thêm -Force nếu muốn ghi đè các model đã có
