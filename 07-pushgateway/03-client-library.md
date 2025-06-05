# Client Library
- Prometheus client libraries allow to push metrics to a Push Gateway efficiently. 
- They support three primary operations that correspond to common HTTP methods for metric manipulation:
    1. A **push** operation that removes any existing metrics for the job and adds the new ones (equivalent to an **HTTP PUT** request).
    2. A **pushadd** operation that updates metrics with the same name while leaving other metrics unchanged (behaving like an **HTTP POST** request).
    3. A **delete** operation that removes all metrics in a specified group (similar to an HTTP **DELETE** request).

- Below is an example demonstrating how to push metrics using the Prometheus client library:
```py
from prometheus_client import CollectorRegistry, Gauge, push_to_gateway

# Create a new registry to hold our metrics
registry = CollectorRegistry()

# Define a gauge metric and register it with the registry
test_metric = Gauge('test_metric', 'This is an example metric', registry=registry)

# Set a value for the metric
test_metric.set(10)

# Push the metric to the Push Gateway
push_to_gateway('user2:9091', job='batch', registry=registry)
```

> [!NOTE]
> - In the example above, the `CollectorRegistry` is used to group all metrics together. 
> - After defining and updating a gauge metric, the `push_to_gateway` function pushes the updated metric to the designated Push Gateway, ensuring that the metrics for the specified job are up to date.
