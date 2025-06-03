# Best Practice

## Naming Convention
Metric names must be written in **snake_case**, meaning all letters are lowercase and words are separated by underscores.

The structure for naming metrics should be:
1. The first term represents the application or library associated with the metric. For example, metrics related to PostgreSQL should start with `postgresql_`.
2. Subsequent terms describe what the metric measures, such as `queue_size`.
3. Always append the unit of measurement (e.g., `seconds`, `bytes`, `meters`) to avoid misinterpretation. This ensures clarity, such as distinguishing between seconds and milliseconds.
4. Use unprefixed base units (like `seconds`, `bytes`, `meters`) rather than their prefixed counterparts (such as `microseconds` or `kilobytes`).
5. Avoid applying special suffixes like `_total`, `_count`, `_sum`, and `_bucket` to custom names except that counter metrics should end with `_total`. Other metric types, including histograms, should not use these suffixes unless required.

> [!TIP]
> library_name_unit_suffix

The standard naming format includes the library name, a description, a unit, and, where applicable, an appropriate suffix.

### Examples of Metric Names
#### Below are some well-crafted examples that adhere to these conventions:

```ini
process_cpu_seconds
http_requests_total
redis_connection_errors
```

- `process_cpu_seconds` uses snake_case, begins with the application/library (`process`), and includes the unit `seconds`.
- `http_requests_total` starts with the relevant component (`http`), describes the metric (`requests`), and appropriately ends with `_total` for a counter metric.
- `redis_connection_errors` clearly identifies the system (Redis) and describes the error type.

#### Conversely, avoid names that deviate from these guidelines:
- Bad Example: `container Docker restarts`
    - Recommendation: Use snake_case and place the library name first. Instead, use `docker_container_restarts`.

- Bad Example: `http_request_sum`
    - Recommendation: Do not use terms like sum which could lead to confusion.

- Bad Example: `nginx_disk_free_kilobytes`
    - Recommendation: Replace `kilobytes` with the base unit `bytes`.

- Bad Example: `dotnet_queue_waiting_time`
    - Recommendation: Always include the unit for clarity (like `seconds`).

---

## What to Instrument
- Choosing what to instrument depends on the system's type and its requirements. 
- Metrics should be tailored to the specific operational context. 
- Generally, there are three main types of applications:

### 1. Online Serving Systems
Online serving systems require immediate responses. They include components such as databases, web servers, and APIs. Common metrics for these systems include:
- Total number of requests or queries
- Number of errors
- Latency measurements
- Number of in-progress requests

### 2. Offline Processing Services
Offline processing services are used where immediate responses are not required. These systems typically perform batch processes involving multiple stages. Metrics to consider include:
- Total amount of work to be done
- Volume of queued work
- Number of work items in progress
- Processing rates
- Errors at various processing stages

### 3. Batch Jobs
Batch jobs are scheduled to run at specific intervals rather than continuously. Because batch jobs do not run continuously, using a Push Gateway is often recommended for effective data collection. Key metrics for batch jobs should include:
- Time spent processing each stage of the job
- Overall runtime of the job
- Timestamp of the last job completion
