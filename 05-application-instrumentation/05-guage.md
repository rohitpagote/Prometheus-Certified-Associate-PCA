# Gauge
In this, we will demonstrate how to create a gauge metric that monitors the total number of active requests—those currently being processed but not yet completed. 

- A gauge metric is similar to a counter; however, it provides additional flexibility by supporting both increment and decrement operations, as well as allowing to set a specific value when needed.

![NOTE]
> A gauge is especially useful for tracking values that can go up and down, such as the number of in-progress requests on your server.

## Gauge Metric Implementation
- Below is the Python code that defines a gauge metric for in-progress requests along with two functions that update the metric during the request lifecycle:

```py
IN_PROGRESS = Gauge('inprogress_requests',
                    'Total number of requests in progress',
                    labelnames=['path', 'method'])

def before_request():
    IN_PROGRESS.labels(request.method, request.path).inc()
    request.start_time = time.time()

def after_request(response):
    IN_PROGRESS.labels(request.method, request.path).dec()
    return response
```

### Implementation Details
1. **Gauge Initialization:**
- The gauge metric, named `inprogress_requests`, is initialized with a description and two labels: `path` and `method`. 
- These labels allow us to differentiate the number of active requests based on the request URL path and HTTP method.

2. **before_request Function:**
- This function is invoked at the beginning of every request. 
- It increments the gauge for the corresponding `method` and `path`, and records the request's start time.

3. **after_request Function:**
- When a request completes, this function is triggered to decrement the gauge, thus reflecting the decrease in active requests.

---

## Monitoring the Gauge Metric
- When querying the metric (for example, with Prometheus), we might receive an output like the following. 
- This output displays the number of active requests per HTTP method and path:

```ini
$ inprogress_requests
inprogress_requests{method="GET",path="/cars"} 3.0
inprogress_requests{method="POST",path="/cars"} 0.0
inprogress_requests{method="POST",path="/boats"} 18.0
inprogress_requests{method="GET",path="/boats"} 7.0
```
