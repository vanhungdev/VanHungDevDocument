# Phân Tích Repo Theo Các Topic Thi

## **Topic 9: Thiết Kế Hướng Đối Tượng**

### ✅ Nội dung đã có trong repo

#### **1. Kiến trúc MVC**

Repo sử dụng mô hình MVC pattern trong ASP.NET Core:

```csharp
// Startup.cs (Lines 50-125)
services.AddControllers(options =>
{
    options.Filters.Add(typeof(HttpGlobalExceptionFilter));
    options.Filters.Add(typeof(ValidateModelStateFilter));
})

app.UseEndpoints(endpoints =>
{
    endpoints.MapControllers();
});
```

- **Controllers**: Xử lý HTTP requests tại `Controllers/*Controller.cs`
- **Routing**: Cấu hình endpoints qua `MapControllers()`

#### **2. Lập trình hướng đối tượng (OOP)**

Code thể hiện rõ các nguyên tắc OOP:

- Sử dụng `class` và `interface`
- Áp dụng Dependency Injection pattern

```csharp
// Startup.cs (Lines 99-103)
services.AddSingleton<INotification, Notification>();
services.AddSingleton<IUserProcess, UserProcess>();
services.AddControllers();
```

#### **3. Design Patterns được áp dụng**

|Pattern|Phân loại|Thể hiện trong code|
|---|---|---|
|**Dependency Injection**|Creational|Đăng ký services qua `IServiceCollection`|
|**Middleware Pipeline**|Behavioral|`app.UseMiddleware<LoggingMiddleware>()`|
|**Filter Pattern**|Behavioral|`HttpGlobalExceptionFilter`, `ValidateModelStateFilter`|

```csharp
// Startup.cs (Lines 111-121)
if (env.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}
app.UseMiddleware<LoggingMiddleware>();
app.UseCustomExceptionHandler();
```


---

## **Topic 10: Nguyên Lý Cơ Bản Ngôn Ngữ Lập Trình**

### ✅ Nội dung đã có trong repo

#### **1. Exception Handling (Xử lý ngoại lệ)**

**Global Exception Filter:**

```csharp
// HttpGlobalExceptionFilter.cs (Lines 20-58)
public void OnException(ExceptionContext context)
{
    if (context.Exception.GetType() == typeof(CustomValidationException))
    {
        // Xử lý validation error
    }
    context.ExceptionHandled = true;
}
```

**Try-Catch trong Controller:**

```csharp
// AuthController.cs (Lines 76-95)
[HttpPost("revoke-token")]
public async Task<IActionResult> RevokeToken()
{
    try
    {
        // Logic xử lý
    }
    catch(Exception ex)
    {
        return Ok(await ResultObject.FailAsync<NullDataType>(
            errorMsg: ex.ToString()
        ));
    }
    return Ok(await ResultObject.OkAsync());
}
```


---

## **Topic 20: Cơ Sở Dữ Liệu – Thiết Kế & Truy Vấn**

### ✅ Nội dung đã có trong repo

#### **1. SQL Schema (Relational Model)**

```sql
-- script.sql (Lines 20-34)
CREATE TABLE [dbo].[ExtendsImage](
    [ID] [int] IDENTITY(1,1) NOT NULL,
    [Image] [nvarchar](500) NOT NULL,
    [AccountID] [int] NOT NULL,
    CONSTRAINT [PK_ExtendsImage] PRIMARY KEY CLUSTERED ([ID] ASC)
)
```

**Các thành phần có:**

- ✅ CREATE TABLE statements
- ✅ PRIMARY KEY constraints
- ✅ FOREIGN KEY relationships (nếu có trong script đầy đủ)

#### **2. ORM Mapping (Entity Framework Core)**

```csharp
// PUBDContext.cs (Lines 9-33)
public partial class PUBDContext : DbContext
{
    public virtual DbSet<User> Users { get; set; }
    
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        if (!optionsBuilder.IsConfigured)
        {
            optionsBuilder.UseSqlServer(
                "Server=34.67.125.213,1433;Database=pubd_db_final;..."
            );
        }
    }
}
```

**Thể hiện:**

- ✅ Relational model mapping qua `DbContext`
- ✅ Entity classes (`DbSet<User>`)
- ✅ SQL queries thông qua LINQ hoặc raw SQL
