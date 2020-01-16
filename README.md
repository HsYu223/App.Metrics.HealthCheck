# App.Metrics.HealthCheck

可以自訂HealthChecks項目,再判斷需要回傳的HealthCheckResult,分別為

1. Healthy   (200)
2. Degraded  (200)
3. UnHealthy (503)

也有 Ping Check, Process Check, HttpGet Check, SQL Check...等,擴充可以使用

[.Framework 4.6.2 範例](Framework462範例.md)

[.NET Core 範例](NETCore範例.md)
