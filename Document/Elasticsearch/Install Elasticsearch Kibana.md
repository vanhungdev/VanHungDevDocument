# Cấu hình Elasticsearch và kibana
- Việc tạo một elasticsearch và kibana giúp chúng ta theo dõi hoạt động của hệ thống và fix bug một cách dễ dàng, tiện lợi

## Phần 1: Cài đặt ELK Stack:  

**Để cài được Elasticsearch, kibna chúng ta cần cài theo các bước sau:**   

1. **Tạo elastic network** - Cần thiết để kết nối elasticsearch với kibna.
2. **Tạo elastic-server container** - Chúng ta sẽ chạy container elasticsearch trước.
3. **Lấy password của tài khoản kibana_system** - Cần lấy password của tk kibana_system vì kibana sẽ không kết nối với tk elastic.
4. **Chạy kibna container** - Chúng ta cần chạy kiban container.
5. **Lấy code từ log kibana và kết nối với elasticsearch** - Chạy thử ở môi trường thật.

**1. Tạo elastic network:**  

 ```bash
  docker network create elastic
 ```
**Bỏ limit size của elasticsearch server:**  
 
 ```bash
sudo vim /etc/sysctl.conf
 ```
Chèn dòng cấu hình `vm.max_map_count=262144` bằng cách nhập i để chuyển vào chế độ chèn, sau đó nhập dòng cấu hình.  

Sau khi cài xong chạy lệnh
 
 ```bash
sudo sysctl -w vm.max_map_count=262144
 ```

**2. Tạo elastic-server container**  

  Tạo elastic-server container:  
 
 ```bash
 docker run --name es01 --restart=always --net elastic -p 9200:9200 -p 9300:9300  -e "discovery.type=single-node" -e "network.host=0.0.0.0"  -e "xpack.security.enabled=true" -e "ELASTIC_PASSWORD=Provanhung77" -e "xpack.security.http.ssl.enabled=false" -t docker.elastic.co/elasticsearch/elasticsearch:8.11.3


 ```

note: sau khi cài xong có thể dùng Provanhung77 để đăng nhập, chúng ta đã cấu hình ở trên

**3. Lấy password của tài khoản kibana_system**  
 
 ```bash
docker exec -it es01 /usr/share/elasticsearch/bin/elasticsearch-reset-password -u kibana_system

 ```
 Nếu vào exec vào rồi hoặc dùng portainer thì chỉ cần 

  ```bash
/usr/share/elasticsearch/bin/elasticsearch-reset-password -u kibana_system

 ```

**4. Chạy kibna container**  
 
 ```bash
  docker run --name kib01 --restart=always --net elastic -p 5601:5601 docker.elastic.co/kibana/kibana:8.11.3 
 ```

**5. Lấy code từ log kibana và kết nối với elasticsearch**  
Lưu ý: cần xem log của kibana và lấy code  
elastic_server là http://elastic-server:9200 hoặc là ip của container name của elasticsearch là được

## Phần 2: Tạo và đánh index elastic: 
Tạo index:
Đoạn code này cần chạy trong mục dev tools

 ```bash
   http://your_ip_address:5601/app/dev_tools#/console
 ```
 
 ```bash
  PUT /name_index1
   {
     "mappings": {
       "properties": {
         "@t": {"type": "date"},
         "@mt": {"type": "text"},
         "@r": {"type": "float"},
         "CorrelationId": {"type": "keyword"},
         "IpAddress": {"type": "ip"},
         "viewResultModel": {"type": "text"},
         "CustomLog": {"type": "text"},
         "Host": {"type": "keyword"},
         "Source": {"type": "keyword"},
         "timestamp": {"type": "date"},
         "DateTime": {"type": "text"},
         "SourceType": {"type": "keyword"},
         "Protocol": {"type": "keyword"},
         "Scheme": {"type": "keyword"},
         "ContentType": {"type": "keyword"},
         "Header": {"type": "object"},
         "LogFolder": {"type": "keyword"},
         "EndpointName": {"type": "keyword"},
         "RequestMethod": {"type": "keyword"},
         "RequestPath": {"type": "keyword"},
         "StatusCode": {"type": "integer"},
         "Elapsed": {"type": "float"},
         "SourceContext": {"type": "keyword"},
         "RequestId": {"type": "keyword"}
       }
     }
   }
 ```

Đẩy dữ liệu vào index:
 
 ```bash
  POST /name_index1/_doc
   {
     "@t": "2023-11-14T06:31:05.4412033Z",
     "@mt": "HTTP {RequestMethod} {RequestPath} responded {StatusCode} in {Elapsed:0.0000} ms",
     "@r": [427.1187],
     "CorrelationId": "276554f2-044c-4c3d-a622-2ee080f1ec05",
     "IpAddress": "::1",
     "viewResultModel": "null",
     "CustomLog": "",
     "Host": "localhost:5001",
     "Source": "1",
     "timestamp": "2023-11-14T13:31:05.4313672+07:00",
     "DateTime": "14-11-2023 13:31:05",
     "SourceType": "USER-LOG",
     "Protocol": "HTTP/1.1",
     "Scheme": "http",
     "ContentType": "text/html; charset=utf-8",
     "Header": [],
     "LogFolder": "bloghung.controllers.homecontroller.index",
     "EndpointName": "/",
     "RequestMethod": "GET",
     "RequestPath": "/",
     "StatusCode": 200,
     "Elapsed": 427.1187,
     "SourceContext": "Serilog.AspNetCore.RequestLoggingMiddleware",
     "RequestId": "0HMV4RD3E53ED:00000001"
   }

 ```

**Cách 2 là call bằng postman**  

Đẩy dữ liệu và thêm index

 ```bash
curl -u "elastic:Provanhung77" -X POST "http://your_ip_address:9200/name_index1/_doc" -H 'Content-Type: application/json' -d '{
    "@t": "2023-11-14T06:31:05.4412033Z",
    "@mt": "HTTP {RequestMethod} {RequestPath} responded {StatusCode} in {Elapsed:0.0000} ms",
    "@r": [427.1187],
    "CorrelationId": "276554f2-044c-4c3d-a622-2ee080f1ec05",
    "IpAddress": "::1",
    "viewResultModel": "null",
    "CustomLog": "",
    "Host": "localhost:5001",
    "Source": "1",
    "timestamp": "2023-11-14T13:31:05.4313672+07:00",
    "DateTime": "14-11-2023 13:31:05",
    "SourceType": "USER-LOG",
    "Protocol": "HTTP/1.1",
    "Scheme": "http",
    "ContentType": "text/html; charset=utf-8",
    "Header": [],
    "LogFolder": "bloghung.controllers.homecontroller.index",
    "EndpointName": "/",
    "RequestMethod": "GET",
    "RequestPath": "/",
    "StatusCode": 200,
    "Elapsed": 427.1187,
    "SourceContext": "Serilog.AspNetCore.RequestLoggingMiddleware",
    "RequestId": "0HMV4RD3E53ED:00000001"
  }'
 ```


**Dữ liệu mẫu để thao tác search**  

```bash
  POST products/_bulk
  {"index":{"_id":4}}
  {"name":"Product 4","desc":"Description 4","price":129,"stock":42,"sale":false,"created_at":"2023-01-04T17:22:08Z"}
  {"index":{"_id":14}}
  {"name":"Product 14","desc":"Description 14","price":290,"stock":4,"sale":true,"created_at":"2023-02-22T00:09:58Z"} 
  {"index":{"_id":15}}
  {"name":"Product 15","desc":"Description 15","price":420,"stock":10,"sale":false,"created_at":"2023-02-26T06:33:29Z"}
  {"index":{"_id":16}}
  {"name":"Product 16","desc":"Description 16","price":340,"stock":12,"sale":false,"created_at":"2023-02-27T04:56:11Z"}
  {"index":{"_id":17}}
  {"name":"Product 17","desc":"Description 17","price":260,"stock":8,"sale":true,"created_at":"2023-02-28T22:13:32Z"}
  {"index":{"_id":18}}
  {"name":"Product 18","desc":"Description 18","price":180,"stock":23,"sale":false,"created_at":"2023-01-01T15:47:24Z"}
  {"index":{"_id":19}}
  {"name":"Product 19","desc":"Description 19 good","price":120,"stock":17,"sale":true,"created_at":"2023-01-05T03:18:45Z"}
  {"index":{"_id":20}}
  {"name":"Product 20","desc":"Description 20 bad","price":220,"stock":11,"sale":false,"created_at":"2023-01-10T12:41:16Z"}
  {"index":{"_id":21}}
  {"name":"Product new york city","desc":"Description new york city","price":220,"stock":11,"sale":false,"created_at":"2023-01-10T12:41:16Z"}

 ```

# KIBANA APM



## Hướng dẫn cài đặt Kibana APM

### 1 Tạo tư mục và cấu hình docker file

```bash
Docker -
       | --- apm-server.yml
       | --- docker-compose.yml

```
cấu hình như sau

file: apm-server.yml
```bash
apm-server:
  host: "0.0.0.0:8200"
  rum:
    enabled: true
  data_streams:
    wait_for_integration: false

output.elasticsearch:
  hosts: ["elasticsearch:9200"]
  username: "my_apm_user"
  password: "YourStrongPassword123"
  ssl.enabled: false

setup.template:
  enabled: true
  
setup.dashboards:
  enabled: true

setup.kibana:
  host: "kibana:5601"
  username: "elastic" 
  password: "Provanhung77"
  ssl.enabled: false

logging:
  level: info
  to_stderr: true

monitoring:
  enabled: false
```

file docker-compose.yml

```bash 

version: '3.8'
services:
  # Elasticsearch Server
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.3
    container_name: elasticsearch
    restart: always
    ports:
      - "9200:9200"
      - "9300:9300"
    environment:
      - discovery.type=single-node
      - network.host=0.0.0.0
      - xpack.security.enabled=true
      - ELASTIC_PASSWORD=Provanhung77
      - xpack.security.http.ssl.enabled=false
      # Bổ sung để tránh lỗi memory
      - "ES_JAVA_OPTS=-Xms1g -Xmx1g"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - elasticsearch_data:/usr/share/elasticsearch/data
    networks:
      - elastic
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9200/_cluster/health"]
      interval: 30s
      timeout: 10s
      retries: 5

  # Kibana
  kibana:
    image: docker.elastic.co/kibana/kibana:8.11.3
    container_name: kibana
    restart: always
    ports:
      - "5601:5601"
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
      - ELASTICSEARCH_USERNAME=kibana_system
      - ELASTICSEARCH_PASSWORD=xWXY94RqAlhG0tnpmzQ=
    networks:
      - elastic
    depends_on:
      elasticsearch:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:5601/api/status"]
      interval: 30s
      timeout: 10s
      retries: 5

  # APM Server
  apm-server:
    image: docker.elastic.co/apm/apm-server:8.11.3
    container_name: apm-server
    restart: always
    ports:
      - "8200:8200"
    environment:
      # Output configuration
      - output.elasticsearch.hosts=["elasticsearch:9200"]
      - output.elasticsearch.username=apm_system
      - output.elasticsearch.password=UMRWQuB9ji5WrV5P3eXy
      - output.elasticsearch.ssl.enabled=false
      
      # APM Server configuration
      - apm-server.host=0.0.0.0:8200
      - apm-server.rum.enabled=true
      
      # Kibana setup configuration
      - setup.kibana.host=kibana:5601
      - setup.kibana.username=elastic
      - setup.kibana.password=Provanhung77
      - setup.kibana.ssl.enabled=false
      
      # Template and dashboard setup
      - setup.template.enabled=true
      - setup.dashboards.enabled=true
      
      # Monitoring configuration  
      - monitoring.enabled=false
      
      # Logging
      - logging.level=info
      - logging.to_stderr=true
      
      # CRITICAL: Add this to fix the precondition check
      - apm-server.data_streams.wait_for_integration=false
      
    volumes:
      - ./apm-server.yml:/usr/share/apm-server/apm-server.yml:ro
    networks:
      - elastic
    depends_on:
      kibana:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8200"]
      interval: 30s
      timeout: 10s
      retries: 5

networks:
  elastic:
    driver: bridge

volumes:
  elasticsearch_data:
    driver: local

```

Lấy mật khẩu kibana:

```bash
docker exec -it elasticsearch /usr/share/elasticsearch/bin/elasticsearch-reset-password -u kibana_system
```

sau khi lấy mật khẩu xong thì cập nhật file và chạy lại docker compose

### 2 Tạo account có quyền ghi file cho apm system 
Phải chạy cả 2

```bash

# 1. Tạo custom role với đủ quyền
curl -X POST "localhost:9200/_security/role/apm_writer" \
  -u elastic:Provanhung77 \
  -H "Content-Type: application/json" \
  -d '{
    "cluster": ["manage_ingest_pipelines", "manage_index_templates", "monitor"],
    "indices": [
      {
        "names": ["apm-*", "traces-*", "logs-*", "metrics-*", ".apm-*"],
        "privileges": ["all"]
      }
    ]
  }'

# 2. Tạo custom role với đủ quyền

  curl -X POST "localhost:9200/_security/user/my_apm_user" \
  -u elastic:Provanhung77 \
  -H "Content-Type: application/json" \
  -d '{
    "password": "YourStrongPassword123",
    "roles": ["apm_writer"],
    "full_name": "Custom APM User"
  }'

```

Check trên kibana tool thử có data chưa

```bash
GET traces-apm*/_search
{
  "size": 0,
  "aggs": {
    "services": {
      "terms": {
        "field": "service.name.keyword"
      }
    }
  }
}



```

### 3 Phải cài APM Integration mới xem được 
http://localhost:5601/app/apm/services

Nếu lỗi thì xóa
```bash
DELETE traces-apm-default
chạy lại docker apm
```





## Bước 4: Test API để tạo data

Mở terminal và chạy vài lệnh này để tạo traces:

```bash
# Gọi API weather forecast
curl http://localhost:5000/weatherforecast
curl http://localhost:5000/weatherforecast
curl http://localhost:5000/health
curl http://localhost:5000/health
```

## Bước 5: Refresh trang APM Services

Sau khi gọi API, refresh trang hoặc đợi vài giây. Bạn sẽ thấy service **"test-opentelemetry-api"** xuất hiện.

## Bước 6: Nếu vẫn không thấy service

Hãy check Data Views:
1. Vào **Stack Management** → **Data Views**
2. Tạo data view mới:
   - **Name**: `APM Traces`
   - **Index pattern**: `traces-apm*`
   - **Timestamp field**: `@timestamp`
3. Tạo thêm:
   - **Name**: `APM Metrics` 
   - **Index pattern**: `metrics-apm*`
   - **Timestamp field**: `@timestamp`

## Bước 7: Kiểm tra trong Dev Tools

Vào **Dev Tools** và chạy:

```json
GET _cat/indices/traces-apm*,metrics-apm*?v

GET traces-apm*/_search
{
  "size": 1,
  "sort": [
    {
      "@timestamp": {
        "order": "desc"
      }
    }
  ]
}
```

Điều này sẽ cho biết có data không và data mới nhất là gì.



```bash
using System;
using Serilog;
using OpenTelemetry.Logs;
using OpenTelemetry.Metrics;
using OpenTelemetry.Trace;
using OpenTelemetry.Resources;
using System.Diagnostics;
using OpenTelemetry.Exporter;
using System.Diagnostics.Metrics;

var builder = WebApplication.CreateBuilder(args);

// ThreadPool config
ThreadPool.SetMinThreads(200, 500);

builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

// Serilog config
builder.Host.UseSerilog((context, configuration) =>
{
    configuration
        .ReadFrom.Configuration(context.Configuration)
        .WriteTo.Console()
        .WriteTo.File("logs/app-.log", rollingInterval: RollingInterval.Day)
        .Enrich.FromLogContext()
        .Enrich.WithProperty("Application", "WeatherForecastApi");
});

builder.Services.AddSingleton<IHttpContextAccessor, HttpContextAccessor>();

// Resource info cho OpenTelemetry
var resourceBuilder = ResourceBuilder.CreateDefault()
    .AddService("test-opentelemetry-api", serviceVersion: "1.0.0")
    .AddAttributes(new Dictionary<string, object>
    {
        ["service.environment"] = "development",
        ["service.instance.id"] = Environment.MachineName,
        ["deployment.environment"] = "local"
    });

// OpenTelemetry setup
builder.Services.AddOpenTelemetry()
    .ConfigureResource(rb => rb
        .AddService("test-opentelemetry-api", serviceVersion: "1.0.0")
        .AddAttributes(new Dictionary<string, object>
        {
            ["service.environment"] = "development",
            ["service.instance.id"] = Environment.MachineName
        }))
    .WithTracing(tracing => tracing
        .AddAspNetCoreInstrumentation(options =>
        {
            options.RecordException = true;
            options.EnrichWithHttpRequest = (activity, request) =>
            {
                activity.SetTag("http.client_ip", request.HttpContext.Connection.RemoteIpAddress?.ToString());
                activity.SetTag("http.user_agent", request.Headers.UserAgent.ToString());
            };
            options.EnrichWithHttpResponse = (activity, response) =>
            {
                activity.SetTag("http.response.size", response.ContentLength);
            };
        })
        .AddHttpClientInstrumentation()
        .AddSource("test-opentelemetry-api")
        .AddOtlpExporter(opt =>
        {
            opt.Endpoint = new Uri("http://localhost:8200/v1/traces");
            opt.Protocol = OtlpExportProtocol.HttpProtobuf;
        })
        .AddConsoleExporter())
    .WithMetrics(metrics => metrics
        .AddAspNetCoreInstrumentation()
        .AddHttpClientInstrumentation()
        .AddRuntimeInstrumentation()
        .AddMeter("test-opentelemetry-api")
        .AddOtlpExporter(opt =>
        {
            opt.Endpoint = new Uri("http://localhost:8200/v1/metrics");
            opt.Protocol = OtlpExportProtocol.HttpProtobuf;
        })
        .AddConsoleExporter());

// Logging export qua OTLP
builder.Logging.AddOpenTelemetry(logging =>
{
    logging.SetResourceBuilder(resourceBuilder);
    logging.AddOtlpExporter(opt =>
    {
        opt.Endpoint = new Uri("http://localhost:8200/v1/logs");
        opt.Protocol = OtlpExportProtocol.HttpProtobuf;
    });
});

// ActivitySource + Meter đăng ký 1 lần
var activitySource = new ActivitySource("test-opentelemetry-api");
var meter = new Meter("test-opentelemetry-api");

// Custom metrics
var forecastCounter = meter.CreateCounter<int>("weather_forecasts_generated", description: "Number of weather forecasts generated");
var processingTime = meter.CreateHistogram<double>("forecast_processing_time_ms", description: "Time taken to process weather forecast");
var apiRequestsCounter = meter.CreateCounter<int>("api_requests_total", description: "Total API requests");
var errorCounter = meter.CreateCounter<int>("api_errors_total", description: "Total API errors");

// Observable gauge - tự động report giá trị
var currentTemp = 20; // Biến để track nhiệt độ hiện tại
var weatherDataGauge = meter.CreateObservableGauge<int>("current_temperature", 
    description: "Current temperature reading",
    observeValue: () => currentTemp);

builder.Services.AddSingleton(activitySource);
builder.Services.AddSingleton(meter);

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

var summaries = new[]
{
    "Freezing", "Bracing", "Chilly", "Cool", "Mild", 
    "Warm", "Balmy", "Hot", "Sweltering", "Scorching"
};

// Weather forecast endpoint với rich tracing
app.MapGet("/weatherforecast", async (ILogger<Program> logger, ActivitySource activitySource) =>
{
    using var activity = activitySource.StartActivity("GenerateWeatherForecast");
    activity?.SetTag("operation.name", "weather_forecast_generation");
    activity?.SetTag("forecast.days", 5);
    activity?.SetTag("service.version", "1.0.0");

    // Count API requests
    apiRequestsCounter.Add(1, new KeyValuePair<string, object?>("endpoint", "/weatherforecast"));

    var sw = Stopwatch.StartNew();
    var forecast = new List<WeatherForecast>();

    try
    {
        // Simulate database connection
        using var dbActivity = activitySource.StartActivity("DatabaseQuery");
        dbActivity?.SetTag("db.system", "postgresql");
        dbActivity?.SetTag("db.operation", "SELECT");
        dbActivity?.SetTag("db.table", "weather_data");
        await Task.Delay(Random.Shared.Next(10, 50)); // Simulate DB query
        dbActivity?.SetStatus(ActivityStatusCode.Ok);

        // Simulate external API call
        using var apiActivity = activitySource.StartActivity("ExternalAPICall");
        apiActivity?.SetTag("http.method", "GET");
        apiActivity?.SetTag("http.url", "https://api.weather.com/v1/current");
        apiActivity?.SetTag("external.service", "weather-service");
        await Task.Delay(Random.Shared.Next(50, 150));
        apiActivity?.SetStatus(ActivityStatusCode.Ok);

        for (int i = 1; i <= 5; i++)
        {
            using var dayActivity = activitySource.StartActivity($"GenerateDay-{i}");
            dayActivity?.SetTag("day.index", i);
            dayActivity?.SetTag("operation.type", "forecast_calculation");
            
            await Task.Delay(Random.Shared.Next(50, 200));

            var temperature = Random.Shared.Next(-20, 55);
            var summary = summaries[Random.Shared.Next(summaries.Length)];
            
            // Update current temperature for gauge metric
            currentTemp = temperature;

            dayActivity?.SetTag("weather.temperature", temperature);
            dayActivity?.SetTag("weather.summary", summary);

            forecast.Add(new WeatherForecast(DateOnly.FromDateTime(DateTime.Now.AddDays(i)), temperature, summary));
            
            // Record metrics
            forecastCounter.Add(1, 
                new KeyValuePair<string, object?>("day", i.ToString()),
                new KeyValuePair<string, object?>("temperature_range", GetTemperatureRange(temperature)));

            logger.LogInformation("Generated forecast for day {Day}: {Temp}°C, {Summary}", i, temperature, summary);
            
            dayActivity?.SetStatus(ActivityStatusCode.Ok);
        }

        sw.Stop();
        processingTime.Record(sw.ElapsedMilliseconds, 
            new KeyValuePair<string, object?>("status", "success"),
            new KeyValuePair<string, object?>("days", "5"));

        activity?.SetTag("forecast.total_processing_time_ms", sw.ElapsedMilliseconds);
        activity?.SetTag("forecast.items_generated", forecast.Count);
        activity?.SetStatus(ActivityStatusCode.Ok, "Weather forecast generated successfully");

        return forecast.ToArray();
    }
    catch (Exception ex)
    {
        errorCounter.Add(1, new KeyValuePair<string, object?>("endpoint", "/weatherforecast"));
        activity?.SetStatus(ActivityStatusCode.Error, ex.Message);
        activity?.RecordException(ex);
        logger.LogError(ex, "Error generating weather forecast");
        throw;
    }
});

// Health check endpoint
app.MapGet("/health", (ILogger<Program> logger) =>
{
    using var activity = activitySource.StartActivity("HealthCheck");
    activity?.SetTag("operation.name", "health_check");
    
    apiRequestsCounter.Add(1, new KeyValuePair<string, object?>("endpoint", "/health"));
    
    var response = new { 
        Status = "Healthy", 
        Timestamp = DateTime.Now,
        Version = "1.0.0",
        Environment = "Development"
    };
    
    activity?.SetTag("health.status", "healthy");
    activity?.SetStatus(ActivityStatusCode.Ok);
    
    logger.LogInformation("Health check performed - Status: {Status}", response.Status);
    
    return Results.Ok(response);
});

// Error simulation endpoint
app.MapGet("/error", (ILogger<Program> logger) =>
{
    using var activity = activitySource.StartActivity("SimulateError");
    activity?.SetTag("operation.name", "error_simulation");
    
    try
    {
        apiRequestsCounter.Add(1, new KeyValuePair<string, object?>("endpoint", "/error"));
        errorCounter.Add(1, new KeyValuePair<string, object?>("endpoint", "/error"));
        
        throw new InvalidOperationException("This is a simulated error for testing APM");
    }
    catch (Exception ex)
    {
        activity?.SetStatus(ActivityStatusCode.Error, ex.Message);
        activity?.RecordException(ex);
        logger.LogError(ex, "Simulated error occurred");
        throw;
    }
});

// Slow endpoint
app.MapGet("/slow", async (ILogger<Program> logger) =>
{
    using var activity = activitySource.StartActivity("SlowOperation");
    activity?.SetTag("operation.name", "slow_operation");
    
    apiRequestsCounter.Add(1, new KeyValuePair<string, object?>("endpoint", "/slow"));
    
    var delay = Random.Shared.Next(2000, 5000);
    activity?.SetTag("operation.delay_ms", delay);
    
    await Task.Delay(delay);
    
    activity?.SetStatus(ActivityStatusCode.Ok);
    logger.LogInformation("Slow operation completed in {Delay}ms", delay);
    
    return Results.Ok(new { Message = "Slow operation completed", DelayMs = delay });
});

app.Lifetime.ApplicationStopping.Register(() =>
{
    activitySource.Dispose();
    meter.Dispose();
});

app.Run();

static string GetTemperatureRange(int temperature)
{
    return temperature switch
    {
        < 0 => "freezing",
        >= 0 and < 15 => "cold",
        >= 15 and < 25 => "mild",
        >= 25 and < 35 => "warm",
        _ => "hot"
    };
}

record WeatherForecast(DateOnly Date, int TemperatureC, string? Summary)
{
    public int TemperatureF => 32 + (int)(TemperatureC / 0.5556);
}

```
