# Observability Standard


## Overview

Observability is the ability to understand the internal state of a service or component by examining its outputs. In software, this means understanding the internal state of a service or component by analyzing its telemetry data, including traces, metrics, and logs. To make a service or component observable, it must be instrumented, meaning the code must emit metrics, logs, and traces. The instrumented data must then be sent to an observability backend.

As our distributed systems scale across multiple regions and services, maintaining a consistent and reliable approach to observability becomes crucial. Without a universal standard for logging, metrics reporting, alerting, dashboards, and tracing, teams risk fragmented insights, delayed incident responses, and increased operational overhead. Establishing a unified observability standard ensures that all components, services, and systems are monitored effectively, enabling faster troubleshooting, improved performance optimization, and a cohesive understanding of system health across the organization. This standard is essential for ensuring that our systems remain resilient, transparent, and scalable.

This observability standard aligns directly with our IT General Controls (ITGC) in the Monitoring & Logging section, ensuring compliance with organizational policies for system integrity and availability. It guarantees that all significant events are logged, monitored, and addressed consistently, supporting audit requirements and promoting operational excellence across our infrastructure.


## Decision

The company has decided to establish an observability standard based on well-known, industry-recognized tools and solutions, avoiding the need for custom-built implementations. Since the majority of our services operate within Kubernetes environments, our observability standard will align with typical cloud-native setups.

This standard will be structured into the following key areas:

* **logging approach**: define a unified approach for log collection and management across all services;
* **metrics reporting approach**: standardize the collection and storage of metrics to monitor system performance and health.
* **alerting standard**: implement a standardized alerting process to ensure timely and effective incident response.
* **dashboards**: create a consistent framework for visualizing key metrics and system states across the organization;
* **tracing approach**: standardize the tracing of requests between distributed services to simplify identification of problem services and bottlenecks or source of errors.


## Logging Approach

The company's logging approach is designed in strict accordance with the [12-factor app methodology](https://12factor.net/logs) to ensure consistency, scalability, and security across all services. All services must output their logs directly to the `STDOUT` and `STDERR` streams. Services must not manage log files or handle the routing and storage of logs. Instead, the execution environment must take full responsibility for capturing, collating, and routing log streams.

During local development, developers may view log streams directly in their terminal to observe application behavior in real-time. In staging and production environments, the execution environment must capture the output streams from all running processes. These logs must be collated with other service logs and routed to designated destinations managed by the infrastructure, ensuring they are available for long-term storage and analysis.

To manage log routing and collection within Kubernetes environments, company has standardized the Unified Logging Layer created on top of the [FluentD - Open source data collector for unified logging layer](https://www.fluentd.org/) and [FluentBit - Super fast, lightweight, and highly scalable logging and metrics processor and forwarder](https://fluentbit.io/). This unified logging layer is deployed with the help of FluentBit DaemonSet across all nodes within each Kubernetes cluster to collect log streams from all running services.

The logging layer is configured to route all logs to the ELK stack—comprising Elasticsearch, Logstash, and Kibana—set up for each Kubernetes cluster. All logs must be collected and processed only within the boundary of a single cluster to ensure isolation, availability, and scalability of the logging layer within each cluster.

Company decided to use the traditional ELK-Stack, consisting of the [ElasticSearch](https://www.elastic.co/elasticsearch) as the long-term storage and indexing engine, enabling efficient retrieval and analysis of logs over time. [Kibana](https://www.elastic.co/kibana) serves as the user interface for accessing and managing these logs.

Access to the logs inside Kibana is controlled via Keycloak's OAuth2 proxy to ensure security. Only authenticated users with valid accounts and known group in the company’s user directory must be granted access, with rights mapped according to user groups defined in the Single Sign-On (SSO) system.

> **WARNING**
>
> Unauthenticated access to logs must not be allowed and strictly prohibited by the current policy.


## Restrictions

In addition to the general logging approach, the company has specific requirements for different types of logs, namely access logs, which collect information about all incoming requests and interactions with the system, and application logs, which collect information about the internal workings of the application for debugging and monitoring.

The company's logging approach mandates that all logs must adhere to a [structured JSON](https://www.json.org/json-en.html) format, therefore be single-lined to ensure consistency and ease of parsing across all systems. Structure of each JSON record must be preserved and each value at given JsonPATH should have the same type and structure between all records to avoid cases where value can be scalar for one request and array / object for another request and this MAY lead to issues with logs parsing and indexing.

Each log entry must include a defined log `level`, which categorizes the severity and purpose of the log message:

* **DEBUG**: detailed debugging information that is useful during development and testing;
* **INFO**: logs that report successful operations and significant events within the application;
* **WARN**: warnings about potentially problematic situations that do not halt application execution;
* **ERROR**: errors that result in crashes or incorrect behavior, but are not critical to the overall application operation;
* **FATAL**: critical errors that require immediate attention and may cause the application to stop functioning.

The volume of logs generated should correspond to the log level. While there is no strict limit on the number of DEBUG logs, collectors may begin throttling log collection if the rate exceeds 3000 logs per second. For other log levels, the volume should remain within a reasonable range to avoid overwhelming the underlying logging layer.

Log entries must comply the retention requirements for their type:

| Data Type | Retention Period | Data Purpose|
|-------------|-------------|-------------|
| Service Logs | Up to 180 days| Monitoring services health, investigating IT incidents |
| Audit Logs | Up to 12 months | Tracking user actions in infrastructure and IT systems with SIEM |
| Access Logs | Up to 3 year | Access logs collected from Cloudfront or Loadbalancer and stored in S3 with comperssion |
| Clairvoyance Metrics| Up to 7 days | Monitoring mobile apps health, investigating IT incidents |
| Prometheus Metrics | Up to 7 days detailed<br/>Up to 2 years downsampled | Monitoring infrastructure and services, creating alerts in case of problems |
| Backups | Up to 30 days | Infrastructure data copy to restore in case of corruption or loss |

Log entries must comply with the requirements for size limits and maximum data loss based on their level:

* logs at INFO level and upper should not exceed 1KB per entry;
* all logs (including DEBUG) must not exceed 3KB per entry, as the log collector enforcing this limit;
* log loss must not exceed 1%;
* logs should be restored within 24 hours if logging stops unexpectedly.

In terms of content, access logs should include specific types of information to enhance their usefulness:

* **startup logs**: detailed logs when the service starts, capturing key initialization steps;
* **configuration load**: logs related to loading application configurations, including sources and parameters (excluding sensitive information like passwords, tokens, and personal data);
* **application termination**: logs that record the termination of the application, including the reasons (e.g., system signals) and runtime duration;
* **internal service connections**:
  * **successful connection**: logs capturing the successful connection to service, including the type (e.g., MySQL, Redis, Kafka) and connection parameters such as the host address, port, database name, etc.;
  * **unsuccessful connection attempts**: logs that detail errors during service connection attempts, including error descriptions and the number of retries.
* **external service interaction**: logs documenting errors when interacting with external services, including the error code and a brief description;
* **event processing errors**: logs that record errors encountered during task execution and event processing, along with their causes.

All logs should also contain the below fields to increase their effectiveness:

* **application logs**:
    * **timestamp**: the time at which the event was logged;
    * **log message**: description of the event or error. (e.g., "Admin has granted rights to the user");
    * **service name**: name of the component generating the log;
    * **user ID**: current user identifier, if applicable (e.g., `user_id`, email, phone):
      * **request ID**: the identifier taken from the request (if applicable) that allows the application and access log to be matched (e.g. X-Amz-Cf-Id);
      * **stack trace**: full stack trace in case of errors or exceptions.
* **access logs**:
    * **timestamp**: the time at which the event was logged;
    * **client IP**: IP address of the user initiating the request (e.g., X-Forwarded-For header's value);
    * **HTTP method**: HTTP method (e.g., GET, POST, PUT);
    * **host (authority)**: Protocol, host, and port;
    * **request URL**: full URL of the request;
    * **response status**: server response code;
    * **request size**: amount of data in bytes sent by the client;
    * **response size**: amount of data in bytes returned to the client;
    * **response time**: time taken to process the request;
    * **user ID**: user identifier, if applicable. For monolith applications, the phone number from query / body. For JWT, the `user_id` from the token.
    * **user agent**: the software used to make the request;
    * **referer**: the URL from which the request was made;
    * **request ID**: the identifier generated on the load balancer side that allows the application and access log to be matched (e.g. X-Amz-Cf-Id);
    * **request body**: data sent by the client.
      For GoLang applications, `lib-go/v2/logger` package must be used, which supports additional features, like changing the application log level in runtime, which may be helpful for debugging problems in runtime on production.

Logs must not contain personal data, secrets, tokens, or passwords. Excluding sensitive information from logs is essential for maintaining security and ensuring compliance with data protection regulations.


### Metrics Reporting Approach

Our company has decided to use the [Prometheus](https://prometheus.io/) for metrics collection and reporting due to its status as the open-source de-facto standard for monitoring within Kubernetes environments. Prometheus is used because it provides powerful querying capabilities, scalable data collection, and native support for dynamic service discovery, which are essential for our distributed applications.

To ensure consistent and effective metrics collection across services in our Kubernetes clusters, all services must adhere to the following standards and practices, described below.


#### Prometheus Integration

Every service running within our platform must expose its metrics in a format compatible with Prometheus. This is typically achieved by providing an HTTP endpoint at a path such as `/metrics` on a separate metrics port. Services that can directly expose metrics via HTTP should implement this approach. This metrics endpoint must be accessible only inside internal network only and conform to the format and conventions expected by Prometheus, see [Prometheus - Exposition Formats](https://prometheus.io/docs/instrumenting/exposition_formats/). This endpoint may be additionally secured according to best practices to prevent unauthorized access.

For services that cannot directly expose metrics, such as cron jobs or batch processes, such services should push their metrics to a Prometheus gateway. This approach is required when scraping metrics is not feasible due to the nature of the service, see [Pushing Metrics](https://prometheus.io/docs/instrumenting/pushing/).

To facilitate automatic metrics collection, our infrastructure utilizes the [Prometheus Operator](https://prometheus-operator.dev/) in conjunction with the `ServiceMonitor` Custom Resource Definitions (CRDs). ServiceMonitor CRDs MUST be configured to specify the target services, the HTTP port, the metrics path (e.g., service-name:8081/metrics), and the interval at which Prometheus should scrape the metrics. It is required that the configuration of these CRDs is accurate and up-to-date to ensure that Prometheus can successfully collect metrics from the appropriate services, that's why observability part is included into the base company chart and simplifies configuration of observability features by rendering required CRD content automatically:

```text
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: some-component
  namespace: some-ns
spec:
  endpoints:
  - interval: 15s
    path: /metrics
    port: metrics
    scrapeTimeout: 10s
  namespaceSelector:
    matchNames:
    - some-ns
  selector:
    matchLabels:
      app.kubernetes.io/name: some-component
```

Prometheus in the company is configured with a minimum 14-day data retention policy. This policy ensures a balance between storage availability and operational needs. For long-term storage and historical analysis, metrics data is exported to configured dedicated long-term storage solution [Thanos - Open source, highly available Prometheus setup with long term storage capabilities](https://thanos.io/).


#### General Requirements

All services running must adhere to the following general requirements for metrics reporting:

* **metric and label naming**: metric and label names must adhere to the [Prometheus metric naming recommendations](https://prometheus.io/docs/practices/naming/). Consistent naming conventions ensure clarity and ease of use when querying metrics across different services;
* **cardinality management**: services must not produce metrics with high cardinality (more than thousands of different values, ideally number should be less than hundreds). High cardinality can significantly increase the storage and processing requirements for metrics, leading to performance degradation in the Prometheus system. Labels used in metrics must not contain dynamic data with an infinite range of values. Examples of such dynamic data include user IDs, session IDs, or timestamps. For services exposed externally, they must not use path or URL component as a metric tag. Without proper control, such service may produce metrics with high cardinality due to path scanning or other external factors.
* **instrumentation based on service type**: services must provide appropriate metrics based on their type. The types of services and corresponding metrics are as follows:
    * **synchronous-processing services (HTTP / gRPC)**: these services must use RED metrics (Rate, Errors, Duration) — to monitor the performance and reliability of the system. Services must categorize response status codes with explicit `status` tag:
        * 5xx — Server errors;
        * 4xx — Client errors;
        * 3xx — Redirects;
        * 2xx — Successful responses.
    - **Asynchronous-Processing Services (Kafka consumers, etc.)**: These services MUST report metrics such as the number of items processed, items in progress, items sent out, and the last time an item was processed (heartbeats are recommended for regular updates).
    - **Batch Jobs**: Metrics MUST include the last time the job succeeded, the duration of each step, the overall runtime, the last completion time, and the total number of jobs processed.


#### GoLang Requirements

All Go services, developed in the company must use the organization's `lib-go/v2/metrix` library for consistent metrics collection.

Following well-known metrics should be used by default where applicable:

* **HTTP server metrics**:
    * `lib_go_http_server_duration{method, path, status, source}`;
    * `lib_go_http_server_requests{method, path, status, source}`;
* **HTTP client metrics**:
    * `lib_go_http_client_duration{target, method, status, path}`;
    * `lib_go_http_client_requests{target, method, status, path}`.
* **MySQL metrics**:
    * `lib_go_mysql_latency{command, result}`;
    * `lib_go_mysql_commands{command, result}`.
* **Redis metrics**:
    * `redis_latency{command, result}`;
    * `redis_commands{command, result}`.


#### Capacity Metrics

Services from the Tier-1/Tier-2 levels (or SMM >=1) should and services from the Tier-3/Tier-4 (or SMM>=3) must report [Utilization/Saturation/Errors (USE)](https://www.brendangregg.com/usemethod.html) metrics that reflect primary information about utilization of all existing service resources to quickly identify existing limits and approaching such limits.

USE stands for:

* **Utilization** — percent time the resource is busy, such as node CPU usage;
* **Saturation** — amount of work a resource has to do, often queue length or node load;
* **Errors** — count of error events.

This method is best for hardware or cloud resources in infrastructure, such as CPU, memory, and network devices. For our cloud computing environment, special cloud controllers are usually in place to limit or throttle running services. This can include hypervisor or container (cgroup) limits for memory, CPU, network, and storage I/O. External hardware or managed services MAY also impose limits, such as for network throughput or CPU credits. Each of these resource limits may be reported and examined with the USE Method, for example, ElastiCache's `EngineCPUUtilization` or Aurora's `CPUUtilization`.


#### Functional and Business Metrics

Services from the Tier-1 / Tier-2 levels (or SMM >=1) should and services from the Tier-3 / Tier-4 (or SMM>=3) must report non-technical, functional metrics that reflect their primary business functions for the purpose of monitoring and alerting. For example, if system is responsible for order creation, relevant metrics should include the number of orders received and the number of orders processed (both successfully and unsuccessfully).

Business metrics, which are aggregations of the work of multiple services, should reflect the overall performance of key business functions.


#### Metrics Reporting — SLO / SLI / SLA

This section establishes the formal standard for defining and monitoring **SLO** (Service Level Objective), **SLI** (Service Level Indicator), and **SLA** (Service Level Agreement) metrics within our services. These metrics are critical for maintaining service quality and ensuring alignment with both technical and business objectives.

* **Service Level Indicator (SLI)**: SLIs must be defined as precise quantitative measures of service performance. SLIs can be technical, such as request latency or error rates, or business-oriented, such as the number of processed orders or registered users;
* **Service Level Objective (SLO)**: SLOs must be set as target values for SLIs. SLOs should be expressed in the form of _SLI ≤ target_ or _lower bound ≤ SLI ≤ upper bound_. SLOs must be aligned with business goals and user expectations;
* **Service Level Agreement (SLA)**: SLAs, where applicable, must formalize the expected level of service and the consequences of failing to meet those expectations. SLA typically assigned per each service tier/Service Maturity Model level.

When defining SLOs, the following standards must be followed:

* SLOs must be aligned with business and product requirements, not merely current system performance;
* SLOs must be simple and straightforward, avoiding complex aggregations that can obscure performance trends;
* SLOs must be realistic and achievable, avoiding absolute targets that are not practical or necessary;
* the number of SLOs should be minimized to cover only the essential aspects of service performance;
* SLOs should be iteratively refined over time based on system behavior and evolving business needs.

Different service types may have distinct SLO and SLI requirements:

* online-serving systems: must implement SLIs based on RED metrics (Rate, Errors, Duration);
* offline processing systems: must track SLIs related to items processed, items in progress, items sent out, and last processing time;
* batch jobs: must include SLIs such as last job success time, duration of each step, overall runtime, and total jobs processed.

SLA per each service is calculated as the ratio of successful events, matching defined SLOs for service to total number of events, providing a value between 0% (complete failure) and 100% (flawless performance).

For example:

* number of successful HTTP requests / total HTTP requests;
* number of gRPC calls completed within target latency / total gRPC requests.

Minimum timeframe for SLA calculation is 1 minute and generally should not be less, however it may be aggregated into 1 hour / 1 day / 1 month / 1 year representation if needed based on information about SLA per minute.

Below is general requirements for minimum SLA level for each Tier (or Service Maturity Model), which must be provided by all company services:

| Service Tier (SMM) | Minimum SLA, %   |
|--------------------|------------------|
| Level 1            | Best-effort mode |
| Level 2            | 95%              |
| Level 3            | 99%              |
| Level 4            | 99.95%           |


#### Alerting Standard

As part of our observability strategy, we have adopted the [Alertmanager](https://prometheus.io/docs/alerting/latest/alertmanager/) as the open-source standard for managing alerts generated from Prometheus metrics. Alertmanager provides a centralized way to handle, group, and route alerts. Applications should define alerting rules in YAML format, and these rules are rendered into Custom Resource Definitions (CRDs) within each Kubernetes cluster using Helm and [Prometheus-Operator - Alerting](https://prometheus-operator.dev/docs/developer/alerting/).

Alertmanager supports multiple alert notification channels, including, but not limited to Slack, and PagerDuty.

In accordance with our incident management scheme:

* all warnings, problems and all incidents must be processed and reported via PagerDuty, which serves as our primary system for handling incident escalation and on-call management;
* warnings, problems and lower-priority alerts may be additionally delivered to dedicated team channels in Slack providing the relevant teams with timely but non-intrusive notifications.

To control alerting configurations, we have introduced several mechanisms in our Helm chart configurations. For example, alerting can be enabled or disabled on a per-cluster basis by specifying the relevant values in the values.yaml files. By default, alerting is enabled across all environments, but if alerts are not required, the configuration MUST be explicitly set in the values file.


##### Configuration

The alerting configuration section is managed under `observability.alertmanagerConfig`. Alerts can be turned on or off by modifying this configuration. By default, this parameter is enabled. However, it must be specified in the `values.yaml` if alerting is not required in a particular cluster:

```yaml
observability:
  alertmanagerConfig:
    enabled: false
```

If multiple notification channels are to be used for a single alert, the `continue` parameter can be enabled to ensure the alert is propagated to all configured channels. By default, this parameter is disabled, and alerts are sent to a single configured channel.

```yaml
observability:
  alertmanagerConfig:
    # ...
    continue: false
```

Prometheus alerting rules, which define the specific metrics and thresholds that trigger alerts, are enabled by default. If you do not plan to roll out alerts, you must explicitly disable this setting by adjusting the `prometheusRule` configuration in `values.yaml`:

```yaml
observability:
  prometheusRule:
    enabled: false
```

YAML files containing alert definitions must be placed in the appropriate directory structure based on the cluster. Alerts that apply globally to all clusters must be stored in the `deployments/aws/alerts directory`, whereas cluster-specific alerts should be stored in the `deployments/aws/<cluster_name>/alerts` folder.

Every alert definition must include the `severity` and `service` labels to ensure that the alerts can be properly categorized and routed. See below an example of defined alert:

```yaml
- alert: Some-service Golden Signals saturation CPU usage from request than 200 percent in 5 minutes
  expr: sum(node_namespace_pod_container:container_cpu_usage_seconds_total:sum_irate{container="some-service"}) / sum(kube_pod_container_resource_requests{job="kube-state-metrics", container="some-service", resource="cpu"}) > 2
  for: 5m
  labels:
    namespace: some-namespace
    service: some-service
    severity: critical
  annotations:
    summary: "Some-service Golden Signals saturation CPU usage from request than 200 percent in 15 minutes"
    runbook_url: "documentary.host/runbook-url"
    grafana_url: "http://grafana.{{ $externalLabels.geo_url }}/d/VDJUBl7Ik/some-service-golden-signals?orgId=1&from=now-1h&to=now"
    kibana_url: "https://kibana.{{ $externalLabels.geo_url }}/"
    description: "Usage of CPU resources greater than 200% from requested during 5 minutes (incident)"
```

Special variable `$externalLabels.geo_url` should be used to form unique URL per each separate Kuberneter cluster instead of hard-coded domain name.


##### Requirements

According to the observability standard, all services must have alerts defined based on existing application metrics to ensure prompt detection and resolution of issues. However, the specific requirements for alerting may vary based on the service tier, as determined by the Service Tiering / Service Maturity Model (SMM).

* **Tier-1 Services (SMM = 1.0)**: for Tier-1 services, which are often in MVP mode or do not have a significant impact on production environments, incidents with critical priority may be disabled. This exception is granted because these services are not expected to cause major disruptions, and therefore, the necessity for critical incident alerts is reduced;
* **Tier-2 Services (SMM >= 2.0)**: starting from Tier-2, services must have well-defined alerts with appropriate `severity` level assigned to each alert. The severity of an alert should be carefully evaluated to ensure that it accurately reflects the potential impact of the issue. Alerts with a critical severity must trigger an incident in production to ensure that any significant issues that could affect production stability are addressed and escalated;
* **Tier-3 and Tier-4 Services (SMM >= 3.0)**: for Tier-3 and Tier-4 services, which typically have a higher SMM score due to their critical role in production environments, additional alerts must be configured based on key service metrics such as Service Level Objectives (SLOs), Service Level Indicators (SLIs), and Service Level Agreements (SLAs). These metrics-based alerts are essential to ensure the stable operation of these services. Alerts should be designed to proactively identify and respond to potential issues before they escalate, thereby maintaining the high availability and reliability required for these critical services.


#### Dashboards

Dashboards are essential for monitoring and maintaining the health of services in production. They provide real-time insights into system performance, resource utilization, and potential issues, enabling teams to quickly detect and respond to anomalies. No service should operate in production without a well-defined set of dashboards to ensure continuous observability.

To standardize our dashboarding approach across the company, we have chosen [Grafana](https://grafana.com) as our primary tool for creating and managing dashboards. Since our services predominantly run in geo-distributed Kubernetes environments across multiple clusters, a traditional, manual approach to dashboard management is impractical. To address this, we have adopted a Dashboards-as-Code strategy. This approach utilizes templated dashboard definitions, making them version-controlled, reproducible, and automatically deployable alongside the services they monitor.

Dashboard(s) must clearly display the current status of the service, including, but not limited to:

* general RED metrics, such as response times, error rates, and request counts;
* general USE metrics, such as saturation metrics, reflecting the usage percentage of service resources and its dependencies;
* state information for key dependencies such as Kafka, Aurora / MySQL, ElastiCache / Redis, or NATS;
* SLO/SLI data to monitor service performance against defined objectives.

Each service and component must have one or more dashboards defined in JSON format. These dashboards are automatically deployed to the relevant Kubernetes cluster during the installation process via our universal Helm chart. Currently, we use the `kiwigrid/k8s-sidecar` container to collect specially annotated `ConfigMaps` containing dashboard definitions and place them into Grafana’s volume. This method ensures that the dashboards remain up-to-date and aligned with each service's deployment.


##### General Requirements

Dashboards are an essential component for ensuring observability and monitoring in production environments. To maintain consistency and reliability, the following architectural decisions and requirements have been established:

* **dashboard enablement**: dashboards must be enabled by default for all services, starting from the Tier-1 (or Level 1 Service Maturity Model). If a service does not use dashboards, this MUST be explicitly configured in the values.yaml file:

  ```yaml
  observability:
    grafanaDashboards:
      enabled: false
  ```
  
* **dashboard files placement**: dashboard JSON files must be placed inside repository of relevant service or component in either the cluster-specific directory `deployments/aws/<cluster_name>/dashboards` or in the shared directory `deployments/aws/dashboards` for dashboards that are common across all clusters. This ensures proper organization and deployment consistency;
* **Grafana folder configuration**: the `grafanaFolder` parameter should be specified to determine the folder in Grafana where dashboards will be stored. If not specified, the folder name will default to the service name:

  ```yaml
  observability:
    grafanaDashboards:
      grafanaFolder: service-name
  ```
  
* **dashboards-as-code**: dashboards must be defined in JSON format and included as part of the Helm chart deployment process.


#### Tracing Approach

Tracing is a critical aspect of our observability strategy, providing visibility into the flow and performance of requests across distributed systems. Effective tracing allows us to diagnose issues, optimize performance, and ensure the reliability of our services.

[Jaeger](https://www.jaegertracing.io/) is our current standard for distributed tracing across services. It allows us to track the lifecycle of a request as it propagates through various microservices, providing insights into latency, bottlenecks, and service dependencies. 

Such traces are collected and aggregated by Jaeger agents running on each node. These agents receive spans from the services and forward them to a centralized Jaeger collector, which then processes and stores the traces for analysis.


##### Requirements

All services must be instrumented to generate traces for all significant operations, capturing both the start and end of each span (operation).

Traces should be initiated at the entry point of a request (e.g., API gateway, frontend service) and must propagate through all relevant services to capture the complete journey of the request.

Services must include sufficient metadata in each trace, such as service name, operation name, and timestamps, to enable detailed analysis.

Tracing should be enabled by default in all environments, including development, staging, and production, to ensure consistent observability.

Sampling strategies should be configured to balance the need for detailed traces with the overhead of collecting and storing large volumes of trace data.


##### Transition to the OpenTelemetry

To align with evolving industry standards and enhance our observability capabilities, we are transitioning to [OpenTelemetry](https://opentelemetry.io/) as our primary tracing solution. OpenTelemetry is an open-source, vendor-neutral standard for distributed tracing and metrics collection, offering greater flexibility and interoperability across different observability tools. Jaeger supports OpenTelemetry standard and this means that we will be able to support our existing tracing solutions and support new OTEL standard for new services and components.

All new services and components must adopt OTEL-compatible tracing client libraries to rely on open-source standard. Existing services should begin migrating from Jaeger-specific instrumentation to OpenTelemetry-based instrumentation. OpenTelemetry provides a set of APIs and SDKs that allow services to generate and export traces in a standardized format.

Consistent with current practice, services must include comprehensive metadata in traces to facilitate deep analysis. This metadata should adhere to OpenTelemetry’s semantic conventions to ensure consistency across services.

Our long-term vision is to fully standardize on OpenTelemetry for tracing across all services, enabling a cohesive and interoperable observability strategy. This approach will allow us to:

* **unify observability data**: integrate tracing with metrics and logs under a single observability framework, providing a comprehensive view of system health and performance;
* **enhance flexibility**: utilize OpenTelemetry’s vendor-neutral approach to seamlessly switch or integrate with different tracing backends as needed;
* **improve interoperability**: align with industry standards, ensuring that our tracing implementation is compatible with a wide range of tools and platforms.
