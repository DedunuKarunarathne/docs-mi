# Exporting Logs to OpenSearch

This guide explains how to export WSO2 MI logs to [OpenSearch](https://opensearch.org/) for centralized log viewing and searching. This is a lightweight setup for basic log management.

!!! tip
    For a complete observability solution with logs, traces, metrics, and pre-built dashboards, see [OpenSearch based Observability]({{base_path}}/observe-and-manage/opensearch-observability/).

## Architecture

```
MI Logs (log4j2) → Fluent Bit → OpenSearch → OpenSearch Dashboards
```

## Prerequisites

- WSO2 MI 4.3 or later
- Docker and Docker Compose

## Step 1 - Create Docker Compose configuration

Create a `docker-compose.yml` file:

```yaml
version: '3'
services:
  opensearch:
    image: opensearchproject/opensearch:2.11.0
    container_name: opensearch
    environment:
      - discovery.type=single-node
      - plugins.security.disabled=true
      - OPENSEARCH_INITIAL_ADMIN_PASSWORD=Observer_123
    ports:
      - "9200:9200"
    networks:
      - opensearch-net

  opensearch-dashboards:
    image: opensearchproject/opensearch-dashboards:2.11.0
    container_name: opensearch-dashboards
    environment:
      - OPENSEARCH_HOSTS=["http://opensearch:9200"]
      - DISABLE_SECURITY_DASHBOARDS_PLUGIN=true
    ports:
      - "5601:5601"
    depends_on:
      - opensearch
    networks:
      - opensearch-net

  fluent-bit:
    image: fluent/fluent-bit:latest
    container_name: fluent-bit
    volumes:
      - ./fluent-bit.conf:/fluent-bit/etc/fluent-bit.conf
      - ./parsers.conf:/fluent-bit/etc/parsers.conf
      - <MI_HOME>/repository/logs:/var/log/wso2mi:ro
    depends_on:
      - opensearch
    networks:
      - opensearch-net

networks:
  opensearch-net:
```

Replace `<MI_HOME>` with the absolute path to your WSO2 MI installation.

## Step 2 - Configure Fluent Bit

Create a `fluent-bit.conf` file:

```ini
[SERVICE]
    Flush        1
    Log_Level    info
    Parsers_File parsers.conf

[INPUT]
    Name              tail
    Path              /var/log/wso2mi/wso2carbon.log
    Tag               mi.logs
    Parser            mi_log
    Refresh_Interval  5
    Buffer_Max_Size   5MB
    Skip_Long_Lines   On

[OUTPUT]
    Name              opensearch
    Match             *
    Host              opensearch
    Port              9200
    Index             wso2-mi-logs
    Suppress_Type_Name On
```

Create a `parsers.conf` file:

```ini
[PARSER]
    Name         mi_log
    Format       regex
    Regex        ^\[(?<time>[^\]]+)\]\s+(?<level>\w+)\s+\{(?<logger>[^}]+)\}\s+-\s+(?<message>.*)$
    Time_Key     time
    Time_Format  %Y-%m-%d %H:%M:%S,%L
```

## Step 3 - Update MI log format (Optional)

For better timestamp parsing, update the log pattern in `<MI_HOME>/conf/log4j2.properties`:

```properties
appender.CARBON_LOGFILE.layout.pattern = [%d{yyyy-MM-dd'T'HH:mm:ss.SSSXXX}] %5p {% raw %}{%c{1}}{% endraw %} - %m %ex %n
```

Update the parser in `parsers.conf` accordingly:

```ini
[PARSER]
    Name         mi_log
    Format       regex
    Regex        ^\[(?<time>[^\]]+)\]\s+(?<level>\w+)\s+\{(?<logger>[^}]+)\}\s+-\s+(?<message>.*)$
    Time_Key     time
    Time_Format  %Y-%m-%dT%H:%M:%S.%L%z
```

## Step 4 - Start services and view logs

1. Start all services:

    ```bash
    docker-compose up -d
    ```

2. Verify OpenSearch is running:

    ```bash
    curl http://localhost:9200
    ```

3. Start the WSO2 MI server and make some API requests to generate logs.

4. Open OpenSearch Dashboards at [http://localhost:5601](http://localhost:5601).

5. Navigate to **Stack Management → Index Patterns** and create an index pattern for `wso2-mi-logs*`.

6. Go to **Discover** to view and search MI logs.

## Searching logs

In OpenSearch Dashboards, you can use Lucene query syntax to search logs:

| Query | Description |
|-------|-------------|
| `level:ERROR` | Find all error logs |
| `level:WARN OR level:ERROR` | Find warnings and errors |
| `logger:org.apache.synapse*` | Logs from Synapse components |
| `message:*timeout*` | Logs containing "timeout" |
| `message:"connection refused"` | Exact phrase match |

## What's next?

- [Configure correlation logs]({{base_path}}/observe-and-manage/classic-observability-logs/monitoring-correlation-logs/) for request tracing
- [OpenSearch based Observability]({{base_path}}/observe-and-manage/opensearch-observability/) for the full solution with traces and metrics
