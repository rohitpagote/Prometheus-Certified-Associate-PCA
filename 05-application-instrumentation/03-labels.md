# Labels
Here, we will see how to modify the application to record the number of HTTP requests per endpoint while still keeping track of the overall record.

- Initially, the application handles requests for two endpoints: `/cars` and `/boats`.
- Let’s start with a basic implementation that uses a global counter which increments on every call.

```py
@app.get("/cars")
def get_cars():
    REQUESTS.inc()
    return ["toyota", "honda", "mazda", "lexus"]

@app.post("/cars")
def create_cars():
    REQUESTS.inc()
    return "Create Car"


@app.get("/boats")
def get_boats():
    return ["boat1", "boat2", "boat3"]

@app.post("/boats")
def create_boat():
    return "Create Boat"
```

- While this method works to keep track of the total number of requests, it does not allow to determine how many requests each endpoint receives. 
- To achieve this, we could create individual counters for every endpoint.

---

## Endpoint-Specific Counters
- One approach is to instantiate separate counters for each endpoint. 
- This method is straightforward, but it requires to manually update queries and configurations each time a new endpoint is added.

```py
CAR_REQUESTS = Counter('requests_cars_total',
                         'Total number of requests for /cars path')
BOATS_REQUESTS = Counter('requests_boats_total',
                           'Total number of requests for /boats path')


@app.get("/cars")
def get_cars():
    CAR_REQUESTS.inc()
    return ["toyota", "honda", "mazda", "lexus"]

@app.post("/cars")
def create_cars():
    CAR_REQUESTS.inc()
    return "Create Car"


@app.get("/boats")
def get_boats():
    BOATS_REQUESTS.inc()
    return ["boat1", "boat2", "boat3"]

@app.post("/boats")
def create_boat():
    BOATS_REQUESTS.inc()
    return "Create Boat"
```

---

## Using Labels for Request Counters
- A more scalable solution is to use **labels**.
- This approach allows us to define a single counter with the flexibility of differentiating request paths using labels.
- When initializing the counter, we specify a list of label names (in this example, we use `path`). 
- Then, while handling a request, call the labels method with the appropriate path before incrementing the counter.

```py
REQUESTS = Counter('http_requests_total',
                   'Total number of requests',
                   labelnames=['path'])


@app.get("/cars")
def get_cars():
    REQUESTS.labels('/cars').inc()
    return ["toyota", "honda", "mazda", "lexus"]

@app.post("/cars")
def create_cars():
    REQUESTS.labels('/cars').inc()
    return "Create Car"


@app.get("/boats")
def get_boats():
    REQUESTS.labels('/boats').inc()
    return ["boat1", "boat2", "boat3"]

@app.post("/boats")
def create_boat():
    REQUESTS.labels('/boats').inc()
    return "Create Boat"
```

- With this setup, we can query the metrics for a specific endpoint by filtering on the `path` label. 
- For example, running the query for the `/cars` endpoint would look like this:

```ini
$ http_requests_total{path="/cars"}
http_requests_total{path="/cars"} 5.0
```

- Similarly, we can retrieve metrics for the `/boats` endpoint or aggregate totals across all paths:

```ini
$ http_requests_total{path="/boats"}
http_requests_total{path="/boats"} 2.0

$ http_requests_total
http_requests_total{path="/cars"} 5.0
http_requests_total{path="/boats"} 2.0

$ sum(http_requests_total)
{} 7.0
```

---

## Extending Labels: Tracking HTTP Methods
- To enhance our monitoring further, we might want to track the HTTP method (e.g., GET or POST) alongside the request path. 
- This requires adding a second label called `method` during counter initialization. 
- When handling a request, provide both the path and the method to the `labels` method.

```py
REQUESTS = Counter('http_requests_total',
                   'Total number of requests',
                   labelnames=['path', 'method'])


@app.get("/cars")
def get_cars():
    REQUESTS.labels('/cars', 'get').inc()
    return ["toyota", "honda", "mazda", "lexus"]

@app.post("/cars")
def create_cars():
    REQUESTS.labels('/cars', 'post').inc()
    return "Create Car"


@app.get("/boats")
def get_boats():
    REQUESTS.labels('/boats', 'get').inc()
    return ["boat1", "boat2"]

@app.post("/boats")
def create_boat():
    REQUESTS.labels('/boats', 'post').inc()
    return "Create Boat"
```
