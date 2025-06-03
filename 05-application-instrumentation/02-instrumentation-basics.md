# Instrumentation Basics
In this tutorial, we will demonstrate how to add instrumentation to your application by building a dummy API using Flask—a robust Python web framework for creating RESTful APIs quickly. 

---

## 1. Creating a Basic Flask Application
- First, create a simple Flask application.
- In Python, we do this by simply importing a Flask package, initialize a Flask instance, and then create an API endpoint.
- For example, any GET request to the `/cars` endpoint will trigger the following function:

```py
from flask import Flask
app = Flask(__name__)

@app.get("/cars")
def get_cars():
    return ["toyota", "honda", "mazda", "lexus"]

if __name__ == '__main__':
    app.run(port='5001')
```

- This code snippet represents the foundation of our API. At this stage, the application does not include any Prometheus instrumentation.

---

## 2. Installing the Prometheus Client Library
- Before adding instrumentation, install the Prometheus client library:

```bash
pip install prometheus_client
```

- Once installed, we can import and use the **Counter** object from the Prometheus client to track your application's metrics. 
- For instance, initialize a counter called `http_requests_total` to record every HTTP request:

```py
from prometheus_client import Counter

REQUESTS = Counter('http_requests_total', 'Total number of requests')

app = Flask(__name__)

@app.get("/cars")
def get_cars():
    return ["toyota", "honda", "mazda", "lexus"]
```

---

## 3. Incrementing the Counter
- To accurately track the number of requests, increment the counter each time the `/cars` endpoint is hit. 
- We can use the `REQUESTS.inc()` method, which increases the counter by 1 for each request. 
- Optionally, we can pass a value to `inc()` if we need a different increment

```py
from prometheus_client import Counter

REQUESTS = Counter('http_requests_total', 'Total number of requests')

app = Flask(__name__)

@app.get("/cars")
def get_cars():
    REQUESTS.inc()
    return ["toyota", "honda", "mazda", "lexus"]
```

---

## 4. Exposing Metrics via a Separate HTTP Server
- Currently, while metrics are recorded, they are not exposed to Prometheus.
- The simplest method to expose these metrics is to start Prometheus’s built-in HTTP server on a separate port (e.g., port 8000)

```py
from prometheus_client import Counter, start_http_server

if __name__ == '__main__':
    start_http_server(8000)
    app.run(port='5001')
```

- With this configuration, our Flask API runs on port `5001` while Prometheus metrics are available at port `8000`.
- We can test the metrics endpoint using:

```bash
$ curl 127.0.0.1:8000
# HELP python_gc_objects_uncollectable_total Uncollectable object found during GC
# TYPE python_gc_objects_uncollectable_total counter
python_gc_objects_uncollectable_total{generation="0"} 0.0
python_gc_objects_uncollectable_total{generation="1"} 0.0
python_gc_objects_uncollectable_total{generation="2"} 0.0
# HELP python_gc_collections_total Number of times this generation was collected
# TYPE python_gc_collections_total counter
python_gc_collections_total{generation="0"} 77.0
python_gc_collections_total{generation="1"} 7.0
python_gc_collections_total{generation="2"} 0.0
# HELP python_info Python platform information
# TYPE python_info gauge
python_info{implementation="CPython",major="3",minor="9",patchlevel="5",version="3.9.5"} 1.0
# HELP http_requests_total Total number of requests
# TYPE http_requests_total counter
http_requests_total 5.0
# HELP http_requests_created Total number of requests
# TYPE http_requests_created gauge
http_requests_created 1.6654499091926205e+09
```
---

## 5. Integrating Metrics with Flask Middleware
- We can also run the API and metrics on the same port.
- We can expose metrics via a dedicated endpoint by integrating Prometheus with Flask middleware. 
- This method uses `make_wsgi_app` from the Prometheus client alongside `DispatcherMiddleware` from Werkzeug

```py
from flask import Flask
from prometheus_client import make_wsgi_app
from werkzeug.middleware.dispatcher import DispatcherMiddleware

app = Flask(__name__)

# Add Prometheus middleware to export metrics at /metrics
app.wsgi_app = DispatcherMiddleware(app.wsgi_app, {
    '/metrics': make_wsgi_app()
})
```

- With this setup, our Flask API is still served on port `5001`, and we can retrieve Prometheus metrics from the `/metrics` endpoint.

---

## 6. Extending the Application with Additional Endpoints
- Next, extend the application by adding several routes that handle different HTTP methods. 
- In this example, endpoints are added to return all cars, retrieve a specific car by ID, create a new car, update car details, and delete a car. 
- In a real-world scenario, we should increment the REQUESTS counter for every request

```py
@app.get("/cars")
def get_cars():
    REQUESTS.inc()
    return ["toyota", "honda", "mazda", "lexus"]


@app.get("/cars/<int:id>")
def get_car():
    REQUESTS.inc()
    return "Single car"


@app.post("/cars")
def create_cars():
    REQUESTS.inc()
    return "Create Car"


@app.patch("/cars/<int:id>")
def update_cars():
    REQUESTS.inc()
    return "Updating Car"


@app.delete("/cars/<int:id>")
def delete_cars():
    REQUESTS.inc()
    return "Deleting Car"
```

- At this point, our fully instrumented Flask API accurately reflects the total number of HTTP requests via the http_requests_total counter. 
- We can access and monitor your metrics either through the standalone Prometheus HTTP server on port `8000` or directly via the `/metrics` endpoint if using the middleware approach.
