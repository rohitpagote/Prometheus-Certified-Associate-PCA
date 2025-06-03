# Histogram/Summary
In this we'll learn how to implement a histogram metric in Python to track the latency and response time for each request in a Flask application. 

## Setting Up the Histogram Metric
- Begin by initializing the histogram metric. 
- In this example, the histogram is named `request_latency_seconds` and includes two label names: `path` and `method`.
- These labels allow to segment metric data based on the request path and HTTP method.

```py
from prometheus_client import Histogram, start_http_server
import time
from flask import request, Flask

app = Flask(__name__)

LATENCY = Histogram('request_latency_seconds', 'Request Latency', labelnames=['path', 'method'])
```

---

## Capturing Request Latency
- To capture request latency, define two functions. 
    - The first function executes before each request, recording the start time. 
    - The second function runs after the request and calculates the latency, which is then recorded by the histogram metric.

```py
def before_request():
    request.start_time = time.time()

def after_request(response):
    request_latency = time.time() - request.start_time
    LATENCY.labels(request.method, request.path).observe(request_latency)
    return response
```

- These callback functions are then integrated into the Flask application using `app.before_request` and `app.after_request`:

```py
if __name__ == '__main__':
    start_http_server(8000)
    app.before_request(before_request)
    app.after_request(after_request)
    app.run(port=5000)
```

### Detailed Explanation
1. **Before Request Callback:**
- The `before_request` function records the current time via `time.time()` when a request is received. 
- This timestamp is stored on the `request` object for later use.

2. **After Request Callback:**
- After the request is processed, the `after_request` function calculates the latency by subtracting the recorded start time from the current time. 
- It then updates the histogram metric using the `observe` method, with the request's HTTP method and path as labels.

This setup provides a robust mechanism to measure the processing time for each request, thereby enabling effective performance monitoring.

---

## Understanding Default Buckets
- When the histogram metric is retrieved—typically via the `/metrics` endpoint—the output might look similar to the example below:
```ini
$ request_latency_seconds
request_latency_seconds_bucket{le="0.005",method="GET",path="/cars"} 0.0
request_latency_seconds_bucket{le="0.01",method="GET",path="/cars"} 0.0
request_latency_seconds_bucket{le="0.025",method="GET",path="/cars"} 0.0
request_latency_seconds_bucket{le="0.05",method="GET",path="/cars"} 1.0
request_latency_seconds_bucket{le="0.075",method="GET",path="/cars"} 3.0
request_latency_seconds_bucket{le="0.1",method="GET",path="/cars"} 3.0
request_latency_seconds_bucket{le="0.25",method="GET",path="/cars"} 4.0
request_latency_seconds_bucket{le="0.5",method="GET",path="/cars"} 6.0
request_latency_seconds_bucket{le="0.75",method="GET",path="/cars"} 6.0
request_latency_seconds_bucket{le="1.0",method="GET",path="/cars"} 8.0
request_latency_seconds_bucket{le="2.5",method="GET",path="/cars"} 8.0
request_latency_seconds_bucket{le="5.0",method="GET",path="/cars"} 8.0
request_latency_seconds_bucket{le="7.5",method="GET",path="/cars"} 8.0
request_latency_seconds_bucket{le="10.0",method="GET",path="/cars"} 8.0
request_latency_seconds_bucket{le="+Inf",method="GET",path="/cars"} 8.0
request_latency_seconds_count{method="GET",path="/cars"} 8.0
```

The Prometheus client library automatically creates default buckets to group latency values. However, these default settings might not be ideal for all use cases.

### Customizing Histogram Buckets
- To tailor the histogram to your application's needs, we can customize the bucket boundaries. 
- Just provide a list of bucket boundaries when initializing the metric. 
- The example below demonstrates how to configure custom buckets:

```py
LATENCY = Histogram(
    'request_latency_seconds',
    'Flask Request Latency',
    labelnames=['path', 'method'],
    buckets=[0.01, 0.02, 0.03, 0.04, 0.05, 0.06, 0.07, 0.08, 0.09, 0.1]
)
```

---

## Configuring a Summary Metric
- Configuring a summary metric is very similar to setting up a histogram. 
The only difference is that you replace `Histogram` with `Summary`. 
- Like the histogram, the summary metric uses the `observe` method after applying the relevant labels.

```py
from prometheus_client import Summary

LATENCY = Summary('request_latency_seconds', 'Flask Request Latency', labelnames=['path', 'method'])

# During the after_request callback
LATENCY.labels(request.method, request.path).observe(request_latency)
```
