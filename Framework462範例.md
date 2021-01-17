# .Framework 4.6.2 範例

## 升級舊網站專案別忘記

```bash
Update-Package -reinstall
```


## 依賴主要套件

```xml
<package id="App.Metrics.Health" version="3.1.0" targetFramework="net462" />
<package id="App.Metrics.Health.Checks.Http" version="3.1.0" targetFramework="net462" />
<package id="App.Metrics.Health.Checks.Network" version="3.1.0" targetFramework="net462" />
<package id="App.Metrics.Health.Checks.Process" version="3.1.0" targetFramework="net462" />
<package id="App.Metrics.Health.Extensions.DependencyInjection" version="3.1.0" targetFramework="net462" />
<package id="Microsoft.Owin.Host.SystemWeb" version="4.0.1" targetFramework="net462" />
```

## Owin Startup

```csharp
public class Startup
{
    public static HttpClient _client = new HttpClient();

    public void Configuration(IAppBuilder app)
    {
        // 如需如何設定應用程式的詳細資訊，請瀏覽 https://go.microsoft.com/fwlink/?LinkID=316888
        app.Map("/health", coreapp =>
        {
            var services = new ServiceCollection();
            var healthBuilder = new HealthBuilder()
                .HealthChecks.RegisterFromAssembly(services)
                .HealthChecks.AddCheck("Healthy Check",
                    () => new ValueTask<HealthCheckResult>(HealthCheckResult.Healthy()))
                .HealthChecks.AddCheck("Degraded Check",
                    () => new ValueTask<HealthCheckResult>(HealthCheckResult.Degraded()))
                .HealthChecks.AddProcessPrivateMemorySizeCheck("Private Memory Size", 1024 * 1024 * 1024)
                .HealthChecks.AddProcessVirtualMemorySizeCheck("Virtual Memory Size", 1024 * 1024 * 1024)
                .HealthChecks.AddProcessPhysicalMemoryCheck("Working Set", 1024 * 1024 * 1024)
                .HealthChecks.AddPingCheck("google ping", "google.com", TimeSpan.FromSeconds(10))
                .HealthChecks.AddHttpGetCheck("weburl", new Uri(""), TimeSpan.FromSeconds(10))
                .BuildAndAddTo(services);

            services.AddHealth(healthBuilder);

            var provider = services.BuildServiceProvider();

            var healthCheckRunner = provider.GetRequiredService<IRunHealthChecks>();
            var healthFormatters = provider.GetRequiredService<IReadOnlyCollection<IHealthOutputFormatter>>();

            coreapp.Run(async context =>
            {
                context.Response.ContentType = "application/json;";
                var healthStatus = await healthCheckRunner.ReadAsync();

                if (!healthStatus.Status.IsHealthy())
                {
                    ChatAlerting(healthStatus.Status.ToString());
                }

                if (healthStatus.Status.IsUnhealthy())
                {
                    context.Response.StatusCode = 503;
                }

                var setting = new JsonSerializerSettings()
                {
                    Converters = new List<JsonConverter>()
                    {
                        new StringEnumConverter()
                    }
                };

                var result = JsonConvert.SerializeObject(
                    healthStatus,
                    Formatting.None,
                    setting);
                await context.Response.WriteAsync(result);

                await Task.FromResult(0).ConfigureAwait(false);
            });
        });
    }

    public void ChatAlerting(string status)
    {
        try
        {
            var token = ConfigurationManager.AppSettings["ChatToken"].ToString();
            var url = "";
            var text = HttpUtility.UrlEncode(JsonConvert.SerializeObject(new JObject(new JProperty("text", Environment.MachineName + " Health Ckech：" + status))));
            var result = _client.GetAsync(url + text).GetAwaiter().GetResult();
        }
        catch (Exception)
        {

        }
    }
}
```
