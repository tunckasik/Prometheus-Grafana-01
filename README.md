
# Hands-on Prometheus & Grafana-01: Prometheus & Grafana Basic

The purpose of this hands-on training is to give students the knowledge of basic operations about Prometheus & Grafana.

## Learning Outcomes

At the end of this hands-on training, students will be able to:

* Learn how to install Prometheus and Grafana
* Learn how to monitor with Prometheus
* Learn how to create a monitoring dashboard with Grafana

## Outline

- Part 1 - Install, configure, and use a simple Prometheus instance
- Part 2 - Monitoring with Prometheus WebUI
- Part 3 - Install, configure, and use a simple Grafana instance
- Part 4 - Creating a monitoring dashboard with Grafana

## Part 1 - Learn install, configure, and use a simple Prometheus instance

- Launch an Amazon EC2 instance with the following settings:

  AMI: "Amazon Linux 2"
  Instance Type: "t2.micro"
  Region: "N.Virginia"
  VPC: "Default VPC"
  Security Group: "Port 22, 3000, 9090, 9100"

- [Download the latest release](https://prometheus.io/download/) of Prometheus for Linux. Select *.linux-*.tar.gz

  ```bash
  wget https://github.com/prometheus/prometheus/releases/download/v2.41.0/prometheus-2.41.0.linux-amd64.tar.gz
  ```
- Extract and run it.

  ```bash
  tar xvfz prometheus-*.tar.gz
  cd prometheus-*
  ```

- Check the basic Prometheus configuration.

  ```bash
  cat prometheus.yml
  ```

  Output:
  ```yaml
  # my global config
  global:
    scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
    evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
    # scrape_timeout is set to the global default (10s).

  # Alertmanager configuration
  alerting:
    alertmanagers:
      - static_configs:
          - targets:
            # - alertmanager:9093

  # Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
  rule_files:
    # - "first_rules.yml"
    # - "second_rules.yml"

  # A scrape configuration containing exactly one endpoint to scrape:
  # Here it's Prometheus itself.
  scrape_configs:
    # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
    - job_name: "prometheus"

      # metrics_path defaults to '/metrics'
      # scheme defaults to 'http'.

      static_configs:
        - targets: ["localhost:9090"]
  ```


- Start Prometheus with the following command:

    ```
    ./prometheus --config.file=prometheus.yml
    ```
- Open the Prometheus web UI from your browser:

    ```
    http://<public ip>:9090
    ```
## Part 2 - Monitoring with Prometheus WebUI
- Check the default metrics of Prometheus:

    ```
    http://<public ip>:9090/metrics
    ```

- Explore various Prometheus expressions and queries using the Prometheus WebUI. Some examples include:

   1. Enter the following expression into the expression console and click "Execute":
        ```
        prometheus_target_interval_length_seconds
        ```
   1. To view only 99th percentile latencies, use this query:
        ```
        prometheus_target_interval_length_seconds{quantile="0.99"}
        ```
   1. To count the number of returned time series, write:
        ```
        count(prometheus_target_interval_length_seconds)
        ```

For more about the expression language, see the [expression language documentation.](https://prometheus.io/docs/prometheus/latest/querying/basics/)

### Monitoring Linux Host Metrics with the Node Exporter

- Install and run the Prometheus Node Exporter:

    ```bash
    wget https://github.com/prometheus/node_exporter/releases/download/v1.3.1/node_exporter-1.3.1.linux-amd64.tar.gz
    tar xvfz node_exporter-*.*-amd64.tar.gz
    cd node_exporter-*.*-amd64
    ./node_exporter
    ```

- Check the metrics exported by the Node Exporter:

    ```bash
    curl http://localhost:9100/metrics
    ```
    or check from your browser:     
    ```
    http://<public ip>:9100/metrics
    ```

- Update the prometheus.yml configuration file to include the Node Exporter metrics as another job as below:

    ```
    # configs for scraping node exporter metrics
    - job_name: node
      static_configs:
        - targets: ['localhost:9100']
    ```

- Stop and run Prometheus again:

    ```bash
    ./prometheus --config.file=./prometheus.yml
    ```
- Check the new metrics on Prometheus web UI.
    | Metric | Meaning |
    | ------ | ------- |
    | rate(node_cpu_seconds_total{mode="system"}[1m])	| The average amount of CPU time spent in system mode, per second, over the last minute (in seconds) |
    | node_filesystem_avail_bytes	                    | The filesystem space available to non-root users (in bytes) |
    | rate(node_network_receive_bytes_total[1m])	    | The average network traffic received, per second, over the last minute (in bytes) |

## Part 3 - Install, configure, and use a simple Grafana instance
- Download and install Grafana from this page: https://grafana.com/grafana/download

    Select **Red Hat, CentOS, RHEL, and Fedora(64 Bit)** part.

    ```
    wget https://dl.grafana.com/enterprise/release/grafana-enterprise-9.3.2-1.x86_64.rpm
    sudo yum install grafana-enterprise-9.3.2-1.x86_64.rpm
    sudo systemctl start grafana-server.service
    ```
- By default, Grafana will be listening on http://localhost:3000. The default login is "admin" / "admin". 
- Open http://<public ip>:3000 and log in with the default credentials("admin"/ "admin").
- Change the default admin password when prompted.

- Add Prometheus as a data source:

    - Click on the gear icon in the left-hand menu to open the Configuration menu.
    - Click on "Data Sources"
    - Click the "Add data source" button.
    - Select "Prometheus" from the list.
    - Fill in the following information:
        - Name: Prometheus
        - URL: http://localhost:9090
        - Access: Server (default)
    - Click "Save & Test" to verify the configuration. If successful, you should see a confirmation message.

## Part 4 - Creating a Prometheus data source in Grafana

### 1. Manual Building a Dashboard
- Create a new dashboard:

    - Click on the "+" icon in the left-hand menu to open the "Create" menu.
    - Click on "Dashboard."
    - Click the "Add new panel" button.

- Configure the new panel:

    - In the "Query" tab, set "Prometheus" as the data source.
    - Enter the following expression into the query editor: 
        ```
        rate(node_cpu_seconds_total{mode="system"}[1m])
        ```
    - Click "Run Query" to view the graph.
    - Click on the panel title and select "Edit" to customize the panel settings.
    - In the "General" tab, set the "Title" to "CPU System Usage."
    - Click "Back" to return to the dashboard.

- Repeat the process to create panels for the other metrics:

    - **node_filesystem_avail_bytes**
    - **rate(node_network_receive_bytes_total[1m])**

    Customize the titles for each panel accordingly.

- Save the dashboard:

    - Click on the floppy disk icon in the top-right corner to open the "Save dashboard" dialog.
    - Enter a name for the dashboard, such as "Node Exporter Metrics."
    - Click "Save."

Your Grafana dashboard is now set up and displaying metrics from the Node Exporter. This setup provides a simple monitoring solution for your Linux host. You can continue to add more panels and explore different metrics to create a comprehensive monitoring solution tailored to your needs.

### 2. Import Pre-Built Dashboard from Grafana.com
- Visit https://grafana.com/grafana/dashboards/ and search for a "node exporter" dashboard. Select one and click "Copy ID to Clipboard".
- On your Grafana web UI (http://<public ip>:3000):
    - Click the Dashboards button.
    - Select Import.
    - Paste the ID of the dashboard (e.g., 12486) into the "Import via grafana.com" field and click "Load".
- Monitor the imported dashboard.

## Conclusion
Congratulations! You have successfully completed this hands-on Prometheus and Grafana project. You have learned how to install, configure, and use simple instances of both tools. You have also monitored metrics using the Prometheus WebUI, created a monitoring dashboard with Grafana, and imported pre-built dashboards from Grafana.com.