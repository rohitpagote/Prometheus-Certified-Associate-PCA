# Console Templates
With Console Templates, we can create custom HTML pages using the Go templating language. 

These pages can embed various metrics, queries, and charts, enabling to design personalized dashboards that prominently display the desired data.

By navigating to `/etc/prometheus/consoles`, we can see several pre-built templates.

```bash
$ ls /etc/prometheus/consoles
drwxr-xr-x  2 user1 user1  4096 Oct  7 12:24  .
drwxr-xr-x  5 user1 user1  4096 Oct 17 22:23  ..
-rw-r--r--  1 user1 user1   616 Oct  7 12:24  index.html.example
-rw-r--r--  1 user1 user1  2675 Oct  7 12:24  node-disk.html
-rw-r--r--  1 user1 user1  3522 Oct  7 12:24  node.html
-rw-r--r--  1 user1 user1  1453 Oct  7 12:24  node-overview.html
-rw-r--r--  1 user1 user1  5783 Oct  7 12:24  prometheus.html
-rw-r--r--  1 user1 user1  4103 Oct  7 12:24  prometheus-overview.html
```

These templates allow to construct individual blocks for different parts of a webpage—such as headers, charts, tables, footers, or sidebars.

---

## Creating a Custom Console Template
To customize our own dashboard, start by navigating to the Prometheus consoles folder and create a new template file named `demo.html` (or use a name of any choice):

```bash
> cd /etc/prometheus/consoles/

> ls
index.html.example  node-cpu.html  node-disk.html  node.html  node-overview.html  prometheus.html  prometheus-overview.html

> sudo vi demo.html
```

Add the following boilerplate:

```html
{{template "head" .}}
{{template "prom_content_head" . }} 
{{template "prom_content_tail" . }}
{{template "tail"}}
```

These template calls add a standard header and footer to your HTML page. To personalize the page, remove any unnecessary sections and integrate your own HTML content. 

---

## Inserting Dynamic Prometheus Data
To display dynamic metric data, we can integrate a PromQL query drilldown.
This feature displays the query result and links to the Prometheus expression browser for further inspection. 

For example, to show the "active memory" metric using the node_memory_Active_bytes query, update your template as follows:

```html
{{template "head" .}}
{{template "prom_content_head" .}}
<h1>Memory details</h1>
active memory: {{template "prom_query_drilldown" (args "node_memory_Active_bytes")}}
{{template "prom_content_tail" .}}
{{template "tail"}}
```
- The drilldown template accepts a single argument, which is the PromQL expression you want to execute.

---

## Adding a Chart
Enhance the dashboard further by adding a chart for visualizing metric trends. To do this, reopen `demo.html` file and insert the following code:

```html
{{template "head" .}}
{{template "prom_content_head" .}}
<h1>Memory details</h1>
active memory: {{template "prom_query_drilldown" (args "node_memory_Active_bytes")}}


<div id="graph"></div>


<script>
new PromConsole.Graph({
    node: document.querySelector("#graph"),
    expr: 'rate(node_memory_Active_bytes[2m])'
})
</script>
{{template "prom_content_tail" .}}
{{template "tail"}}
```

In this enhanced template:
- The `<div>` with the ID "graph" acts as a placeholder for the chart.
- A JavaScript block creates a new graph using `new PromConsole.Graph({...})` with the following parameters:
    - The `node` parameter locates the `<div>` container using `document.querySelector("#graph")`.
    - The `expr` parameter supplies the PromQL query `rate(node_memory_Active_bytes[2m])` to fetch the metric data.

After saving the file, accessing demo.html via your Prometheus server will display your custom dashboard. 
