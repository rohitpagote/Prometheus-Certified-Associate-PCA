# SLI/SLO/SLA
- When designing a robust system or application, it is essential for teams to define clear, measurable targets and goals. 
- These targets help customers & end users quantify the level of reliability they should come to expect from a service.
- Example: "Application should have 97% uptime in a rolling 30 day window"

--- 

## Service Level Indicator(SLI)
- A Service Level Indicator (SLI) is a quantitative metric that evaluates a specific aspect of service performance. 
- In simpler terms, an SLI measures how well a service operates based on key performance indicators.
- Common SLIs include:
    - Request latency
    - Error rates
    - Saturation or throughput
    - Availability (e.g., ensuring 99% uptime)
- It is important to understand that not every metric qualifies as an SLI. 
- The most effective SLIs closely mirror the user’s experience with the service. 
- For example, high CPU usage or memory consumption may be a concern from a technical perspective but do not necessarily translate into a degraded user experience. Instead, metrics such as response time and error rates serve as more accurate indicators of service quality from the customer's viewpoint.

---

## Service Level Objective (SLO)
- A Service Level Objective (SLO) represents the target value or acceptable range for an SLI. 
- For example, if an SLI reads latency, an SLO might be set to keep latency below 100 milliseconds. Similarly, an availability SLO could state that the service must achieve 99.9% uptime. 
- Essentially, an SLO provides a clear, measurable goal aligned with the customer experience.
- SLOs should be directly related to the customer experience. 
- The main purpose of the SLO is to quantify reliability of a product to a customer

---

## Service Level Agreement (SLA)
- A Service Level Agreement (SLA) is a formal contract between a vendor and a user that commits to meeting a specific SLO. 
- SLAs are typically legally binding, and failure to meet the agreed-upon targets may result in financial penalties or other corrective measures.

---

> [!NOTE]
> In summary, SLIs are used to measure the quality of a service, SLOs set the target performance levels for these indicators, and SLAs formalize these targets in a binding agreement. Together, they play a critical role in ensuring that customers receive a consistent and high-quality service.
