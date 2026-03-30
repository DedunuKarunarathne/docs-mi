# OpenSearch based observability

The WSO2 Observability Solution provides monitoring and analytics capabilities for WSO2 Micro Integrator based on open-source products and standards such as [OpenSearch](https://opensearch.org/), Fluent Bit, and OpenTelemetry.

The solution includes:

- **Log analytics** - Centralized log collection and analysis
- **Distributed tracing** - End-to-end request tracing across services
- **Metrics analytics** - Performance metrics and dashboards
- **Pre-built dashboards** - Ready-to-use Integration logs and metrics dashboards

## Architecture

The observability solution uses the following components:

| Component | Description |
|-----------|-------------|
| **OpenSearch** | Stores and indexes logs, traces, and metrics data |
| **OpenSearch Dashboards** | Provides visualization and analytics interface |
| **Fluent Bit** | Collects and ships log files to OpenSearch |
| **Data Prepper** | Processes OpenTelemetry data (traces and metrics) |

The architecture follows this pattern:

```
MI Logs (log4j2) → Fluent Bit → OpenSearch
MI Traces/Metrics (OTLP) → Data Prepper → OpenSearch
```

## Prerequisites

=== "Kubernetes"

    - Kubernetes cluster (minimum 8 vCPUs, 12 GB memory recommended)
    - [Helm](https://helm.sh/docs/intro/install/) installed
    - kubectl configured

=== "VM/Docker"

    - Docker and Docker Compose
    - For VM deployment: [Puppet](https://www.puppet.com/docs/puppet/8/install_agents) installed

## Deploying the Observability Solution

=== "Kubernetes"

    1. Clone the observability resources repository:

        ```bash
        git clone https://github.com/wso2/observability-resources.git
        cd observability-resources
        ```

    2. Deploy the observability solution:

        ```bash
        cd observability/k8s/
        sh deploy-observability.sh
        ```

    3. Port forward the OpenSearch Dashboards service:

        ```bash
        kubectl port-forward svc/opensearch-dashboards 5601:5601 -n observability
        ```

    4. Access the dashboard at [http://localhost:5601](http://localhost:5601) using default credentials:
        - Username: `admin`
        - Password: `admin`

=== "VM/Docker"

    1. Clone the observability resources repository:

        ```bash
        git clone https://github.com/wso2/observability-resources.git
        cd observability-resources
        ```

    2. Deploy all components locally:

        ```bash
        cd observability/vm/
        sh deploy.sh local
        ```

        !!! note
            On Linux, run with sudo: `sudo -E sh deploy.sh local`

        You can also deploy individual components:

        - `sh deploy.sh opensearch` - OpenSearch
        - `sh deploy.sh opensearch-dashboards` - OpenSearch Dashboards  
        - `sh deploy.sh fluentbit` - Fluent Bit
        - `sh deploy.sh data-prepper` - Data Prepper

    3. Access the dashboard at [http://localhost:5601](http://localhost:5601) using default credentials:
        - Username: `admin`
        - Password: `Observer_123`

## Configuring the Micro Integrator

To publish observability data from MI to the solution, configure the following settings.

### Step 1 - Configure deployment.toml

Add the following configuration to `<MI_HOME>/conf/deployment.toml`:

```toml
[mediation]
flow.statistics.enable = true
flow.statistics.capture_all = true

[analytics]
enabled = true
publisher = "log"
id = "wso2mi_server"
prefix = "SYNAPSE_ANALYTICS_DATA"
api_analytics.enabled = true
proxy_service_analytics.enabled = true
sequence_analytics.enabled = true
endpoint_analytics.enabled = true
inbound_endpoint_analytics.enabled = true

[opentelemetry]
enable = true
logs = true
type = "otlp"
url = "http://<DATA_PREPPER_HOST>:4317"
```

Replace `<DATA_PREPPER_HOST>` with:

- **Kubernetes**: `data-prepper.observability.svc.cluster.local`
- **VM/Docker (local)**: `localhost` or the Data Prepper host IP

### Step 2 - Configure log4j2.properties

Update `<MI_HOME>/conf/log4j2.properties` with the following configurations.

1. Update the CARBON_LOGFILE appender pattern for OpenSearch-compatible timestamps:

    ```properties
    appender.CARBON_LOGFILE.layout.pattern = [%d{yyyy-MM-dd'T'HH:mm:ss.SSSXXX}] %5p {% raw %}{%c{1}}{% endraw %} %X{Artifact-Container} - %m %ex %n
    ```

2. Add the analytics appender:

    ```properties
    appender.MI_ANALYTICS_LOGFILE.type = RollingFile
    appender.MI_ANALYTICS_LOGFILE.name = MI_ANALYTICS_LOGFILE
    appender.MI_ANALYTICS_LOGFILE.fileName = ${sys:carbon.home}/repository/logs/synapse-analytics.log
    appender.MI_ANALYTICS_LOGFILE.filePattern = ${sys:carbon.home}/repository/logs/synapse-analytics-%d{MM-dd-yyyy}-%i.log
    appender.MI_ANALYTICS_LOGFILE.layout.type = PatternLayout
    appender.MI_ANALYTICS_LOGFILE.layout.pattern = [%d{yyyy-MM-dd'T'HH:mm:ss.SSSXXX}] [%X{ip}-%X{host}] [%t] %5p %c{1} %m%n
    appender.MI_ANALYTICS_LOGFILE.policies.type = Policies
    appender.MI_ANALYTICS_LOGFILE.policies.time.type = TimeBasedTriggeringPolicy
    appender.MI_ANALYTICS_LOGFILE.policies.time.interval = 1
    appender.MI_ANALYTICS_LOGFILE.policies.time.modulate = true
    appender.MI_ANALYTICS_LOGFILE.policies.size.type = SizeBasedTriggeringPolicy
    appender.MI_ANALYTICS_LOGFILE.policies.size.size = 1000MB
    appender.MI_ANALYTICS_LOGFILE.strategy.type = DefaultRolloverStrategy
    appender.MI_ANALYTICS_LOGFILE.strategy.max = 10
    ```

3. Add the analytics logger:

    ```properties
    logger.ANALYTICS_LOGGER.name = org.wso2.micro.integrator.analytics.messageflow.data.publisher.publish.elasticsearch.ElasticStatisticsPublisher
    logger.ANALYTICS_LOGGER.level = DEBUG
    logger.ANALYTICS_LOGGER.additivity = false
    logger.ANALYTICS_LOGGER.appenderRef.MI_ANALYTICS_LOGFILE.ref = MI_ANALYTICS_LOGFILE
    ```

4. Include the appender and logger in the respective lists:

    ```properties
    appenders = CARBON_CONSOLE, CARBON_LOGFILE, ..., MI_ANALYTICS_LOGFILE
    loggers = AUDIT_LOG, ANALYTICS_LOGGER, ...
    ```

### Step 3 - Configure Fluent Bit to read MI logs

=== "Kubernetes"

    Add the `component: "wso2mi"` and `app: "<app-name>"` labels to your MI Kubernetes deployment to enable log collection.

=== "VM/Docker"

    Update the `mi_logs_path` field in `<observability-resources>/observability/vm/puppet/code/environments/production/modules/fluentbit/manifests/params.pp`:

    ```puppet
    $mi_logs_path = "/path/to/mi/repository/logs"
    ```

    Restart Fluent Bit after updating the configuration.

## Viewing Observability Data

After configuring MI and generating some traffic:

1. Open the OpenSearch Dashboard at [http://localhost:5601](http://localhost:5601).

2. Navigate to **Dashboards** and select **Integration logs dashboard** to view MI log analytics.

3. Navigate to **Observability → Traces** to view distributed tracing data.

4. Navigate to **Dashboards → Integration metrics dashboard** to view performance metrics.

## What's Next?

- Explore the [WSO2 Observability Resources](https://github.com/wso2/observability-resources) repository for sample deployments and advanced configurations
- Learn about [configuring correlation logs]({{base_path}}/observe-and-manage/classic-observability-logs/configuring-correlation-logs/) for request tracking
```

!!! tip "See also"
    - For a production-ready observability setup with dashboards, see the [WSO2 Observability Resources](https://github.com/wso2/observability-resources) repository.
    - For OpenTelemetry-based log correlation with traces, see [OpenTelemetry Based Logs]({{base_path}}/observe-and-manage/classic-observability-logs/opentelemetry-logs/).
