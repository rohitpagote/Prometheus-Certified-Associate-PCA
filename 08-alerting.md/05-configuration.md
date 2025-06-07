# Alertmanager Configuration
The Alertmanager configuration (typically stored in `alertmanager.yml`) is divided into three main sections: 
- Global configuration
- Routing rules
- Receivers

---

# Global Configuration
- The global configuration sets **default parameters for the entire Alertmanager setup**.
- This structure mirrors the global configuration approach in Prometheus.
- Below is an example of a global configuration block with routing rules and receivers defined:
```yml
global:
  smtp_smarthost: 'mail.example.com:25'
  smtp_from: 'test@example.com'
route:
  receiver: staff
  group_by: ['alertname', 'job']
  routes:
    - match_re:
        job: (node|windows)
      receiver: infra-email
    - matchers:
        job: kubernetes
      receiver: k8s-slack
receivers:
  - name: k8s-slack
    slack_configs:
      - channel: '#alerts'
        text: 'https://exampl.com/alerts/{{ .GroupLabels.app }}'
```

Key components in this configuration include:
- **Global Settings**: Define SMTP parameters and other default configurations.
- **Route Section**: Establishes matching rules to determine which alerts are sent to which receivers.
- **Receivers Section**: Contains notifier configurations (e.g., Slack, email) that manage how alerts are delivered.

---

# Routing Section
- The routing section assigns alerts to specific receivers based on matching criteria. 
- Routes can use either `matchers` for direct label comparisons or `match_re` for regular expression-based matching.

> [!NOTE]
> Use matchers for exact label value comparisons. Leverage match_re when you require more flexible, regular expression-based matching.

- For example, the following route configuration uses both methods:
```yml
route:
  routes:
    - match_re:
        job: (node|windows)
      receiver: infra-email
    - matchers:
        job: kubernetes
        severity: ticket
      receiver: k8s-slack
```

## Default (Fallback) Route
- We can define a default or fallback route at the top level of routing configuration.
- These routes catch any alerts that do not match more specific rules. 
- In the example below, alerts that do not match any nested routes are grouped by `alertname` and `job`, then sent to the receiver named `staff`:
```yml
route:
  receiver: staff
  group_by: ['alertname', 'job']
  routes:
    - match_re:
        job: (node|windows)
      receiver: infra-email
    - matchers:
        job: kubernetes
      receiver: k8s-slack
```

---

# Nested Routes (Subroutes)
- Alertmanager supports nested routes (subroutes) to allow for additional filtering. 
- With nested routes, a parent route can first match alerts with a specific label (such as `job`), and subroutes can further refine the selection based on additional criteria, like `severity`.
- Consider the following configuration. Alerts with the label `job: kubernetes` are routed to `k8s-email` by default; however, if an alert also includes `severity: pager`, it is forwarded to `k8s-pager`:
```yml
route:
  routes:
    - matchers:
        job: kubernetes
      receiver: k8s-email
      routes:
        - matchers:
            severity: pager
          receiver: k8s-pager
```
Processing steps include:
- The alert is first evaluated against the parent route (matching `job: kubernetes`).
- If the alert has the label `severity: pager`, the subroute is matched and sent to the `k8s-pager` receiver.
- Alerts missing the extra `severity` label default to the parent route’s receiver (`k8s-email`).

---

# Managing Multiple Teams with Subroutes
- We can further refine routing for multiple teams by using subroutes. 
- This approach ensures that alerts reach the appropriate team's notifiers. 
- The following configuration demonstrates how to set up parent routes based on a team label, with subroutes handling alert severity:
```yml
route:
  routes:
    # Database team configuration
    - match:
        team: database
      receiver: database-pager
      routes:
        - match:
            severity: page
          receiver: database-pager
        - match:
            severity: email
          receiver: database-email
    # API team configuration
    - match:
        team: api
      receiver: api-pager
      routes:
        - match:
            severity: page
            env: dev
          receiver: api-ticket
        - match:
            severity: page
          receiver: api-pager
        - match:
            severity: ticket
          receiver: api-ticket
```
In this configuration:
- Alerts for the "database team" default to `database-pager`, unless further specifications direct them to `database-email`.
- Alerts for the "API team" default to `api-pager`, with additional subroutes managing alerts based on environment or severity.

---

# Reloading the Alertmanager Configuration
- After making changes to your `alertmanager.yml` file, Alertmanager does not automatically reload the updated configurations. 
- We must take one of the following actions to apply changes:
    - Restart the Alertmanager process.
    - Send a SIGHUP signal. For example:
```bash
sudo killall -HUP alertmanager
```
    - Send an HTTP POST request to the /-/reload endpoint.

---

# Matching Multiple Routes
- By default, Alertmanager stops processing routes after the first match is found. In the example below:
```yml
route:
  routes:
    - receiver: alert-logs
    - matchers:
        job: kubernetes
      receiver: k8s-email
```
- If an alert matches the first route (`alert-logs`), it will not be evaluated for further matches with the Kubernetes-specific rule. 
- To allow an alert to trigger multiple receivers, use the `continue` property on a route. 
- Setting `continue: true` instructs Alertmanager to evaluate subsequent routes even after a match.

---

# Alert Grouping
- Alert grouping consolidates multiple alerts into a single notification based on specified label values and it is done by using `group_by` field.
- For example, grouping alerts by team ensures that alerts for the same team are sent together:
```yml
route:
  receiver: fallback-pager
  group_by: [team]
routes:
  - match:
      team: infra
    group_by: [region, env]
    receiver: infra-email
    routes:
      - match:
          severity: page
        receiver: infra-pager
```
In this configuration:
- The default route groups alerts based on the `team` label.
- A nested route for the infrastructure team further groups alerts by `region` and `env`.
- Child routes inherit the grouping rules unless they define their own `group_by` values.
