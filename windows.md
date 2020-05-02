---

copyright:
  years:  2018, 2020
lastupdated: "2020-02-12"

keywords: Sysdig, IBM Cloud, monitoring, windows

subcollection: Sysdig

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:pre: .pre}
{:table: .aria-labeledby="caption"}
{:codeblock: .codeblock}
{:tip: .tip}
{:download: .download}
{:important: .important}
{:note: .note}


# Monitoring a Windows environment
{: #windows}

Windows support with Prometheus

{{site.data.keyword.mon_full_notm}} is a third-party cloud-native, and container-intelligence management system that you can include as part of your {{site.data.keyword.cloud_notm}} architecture. Use it to gain operational visibility into the performance and health of your applications, services, and platforms. It offers administrators, DevOps teams and developers full stack telemetry with advanced features to monitor and troubleshoot, define alerts, and design custom dashboards. {{site.data.keyword.mon_full_notm}} is operated by Sysdig in partnership with {{site.data.keyword.IBM_notm}}.
{:shortdesc}

Complete the following steps to configure the following Windows images to send metrics to a Sysdig instance:
* Windows Server 2019 Standard Edition (64 bit)
* Windows Server 2016 Standard Edition (64 bit)
* Windows Server 2012 Standard Edition (64 bit)
* Windows Server 2012 R2 Standard Edition (64 bit)


To monitor a Windows system with {{site.data.keyword.mon_full_notm}}, complete the following steps:

## Step 1. Configure the Prometheus WMI exporter
{: #windows_step1}

Configure the Prometheus WMI exporter to collect Windows system metrics.

The Prometheus WMI exporter runs as a Windows service. You configure the metrics that you want to monitor by enabling collectors. 

The following collectors are supported:
* CPU
* Computer system metrics (cs)
* Disk metrics
* Network interface metrics


Comnplete the following steps to configure the Prometheus WMI exporter in your Windows system:

1. Login to your Windows system, for example, you can connect via remore desktop (RDP).

2. [Download the Prometheus exporter ![External link icon](../../icons/launch-glyph.svg "External link icon")](https://github.com/martinlindhe/wmi_exporter/releases){:new_window}.

3. Identify the collectors that include data for the metric data that you want to send to Sysdig.  

4. Run the `wmi_exporter` and configure the collectors that you want to enable.

    ```
    .\wmi_exporter-0.10.2-amd64.exe --collectors.enabled COLLECTORS 
    ```
    {: codeblock}

    Where 
    * COLLECTORS` indicates the list of connectors that you want to configure.

    For example, to collect computer system metrics (cs), CPU metrics, disk metrics and network interface I/O metrics, see the following example:

    ```
    .\wmi_exporter-0.10.2-amd64.exe --collectors.enabled "cpu,cs,logical_disk,net"
    ```
    {: codeblock}


## Step 2. Configure the Prometheus Blackbox exporter
{: #windows_step2}

Configure the Prometheus Blackbox exporter to get information about the availability of a Windows system.

The Prometheus Blackbox exporter can be run as an application or a docker container from a Linux system in conjunction with the Sysdig agent.

The [Blackbox exporter](docs/observability-monitoring?topic=observability-monitoring-blackbox-exporter) 


## Step 3. Configure network settings
{: #windows_step3}

Complete the following steps:
1. Enable the Windows firewall to allow access to `wmi_exporter-0.10.2-amd64.exe`.

2. [Optional] Update the VPC rules

    If you use private endpoints, add an inbound rule to the security group for port `9182` with `source type = Security Group` and choose the security group for the Windows system.


## Step 4. Choose the method to collect metrics
{: #windows_step4}


### Option 1: Run a Sysdig agent on a Linux system and remotely scrape the Windows endpoint
{: #windows_step4_1}

Complete the following steps:

1. Update the `/opt/draios/etc/dragemt.yaml` to [enable remote scraping ![External link icon](../../icons/launch-glyph.svg "External link icon")](https://docs.sysdig.com/en/collecting-prometheus-metrics-from-remote-hosts.html){:new_window}. 

    See the following sample configuration to enable scraping for each of the Windows systems in your environment:

    ```
    prometheus:
    enabled: true
    interval: 30
    log_errors: true
    max_metrics: 3000
    max_metrics_per_process: 3000
    max_tags_per_metric: 20
    remote_services:
    - windows_test_01:
        always: true
        conf:
            url: "http://10.240.0.5:9182/metrics"
        tags:
            service: windows
            windows_hostname: windows-test-01
    ```
    {: codeblock}

2. [Install the Sysdig agent on a Linux node](https://cloud.ibm.com/docs/Monitoring-with-Sysdig?topic=Sysdig-config_agent#config_agent_linux).

    `curl -sL https://ibm.biz/install-sysdig-agent | sudo bash -s -- -a <access key> -c ingest.private.<region>.monitoring.cloud.ibm.com --collector_port 6443 --secure true -ac "sysdig_capture_enabled: false"`

    Where

    `<access key>` is the Sysdig access key for the agent.

    `<region>` is the region where the Sysdig instance is available.
    You can collect a maximum of 3000 time series per Sysdig Linux agent. If you need to collect more than 3000 time series for all your Windows systems, you need more than one Linux agent.
    {: important}
    
3. Configure the Sysdig agent to reduce the number of metrics that are collected by the Windows wmi_exporter. 

    You can either remove the metrics for the collector itself or other metrics that you do not wish to collect by configuring the `metrics_filter` section.

    For example, to remove go metrics and the logical disk request metrics, you can set the section in the following way:

    ```
    metrics_filter:
      - exclude: go_*
      - exclude: wmi_logical_disk_requests_queued
    ```
    {: codeblock}



### Option 2: Use the Prometheus remote-write capabilities to push the metrics from the Windows system by running Prometheus as a client collector on Windows
{: #windows_step4_2}

Complete the following steps:

1. Download the Prometheus monitoring system and time series database. [Dwonload prometheus-2.15.2.windows-amd64.tar.gz](https://prometheus.io/download/) 

2. Unzip the file `prometheus-2.15.2.windows-amd64.tar.gz`. 

3. Edit the `prometheus.yml` file. For example, you can edit it with notepad. 

4. Configure the `prometheus.yml` as follows:
    
    Modify the file as follows:

    ```yaml
    # Global configuration
    global:
      scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
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

    # Scrape configuration that contains one endpoint to scrape:
    scrape_configs:
      # The job name is added as a label `job=<job_name>` to any timeseries scraped from this configuration.
      - job_name: 'prometheus'

        # metrics_path defaults to '/metrics'
        # scheme defaults to 'http'.

        static_configs:
        # - targets: ['localhost:9090']
    ```
    {: codeblock}

    For example, for a Windows 2012 R2 image, see the following example for the `scrape_configs` section:

    ```yaml
    # A scrape configuration containing exactly one endpoint to scrape:
    scrape_configs:
      # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
      - job_name: 'prometheus'

        static_configs:
        - targets: ['localhost:9182']

          labels:
            instance: "window 2012 R2"
            region: "us-south"
    ```
    {: codeblock}


    Then, add the following sections, **remote_write** and **bear_token**.

    ```yaml
    remote_write:
       - url: "ENDPOINT/api/prometheus/write"
  
        bearer_token_file: C:\Users\Administrator\prom\BEARER_TOKEN

        write_relabel_configs:
          # Drop forwarding the metrics generated by the exporter that are not supported
          - source_labels: ["__name__"]
            regex: "^wmi_(.*)"
            action: keep
    
          - source_labels: ["__name__"]
            regex: "^wmi_(.*)"
            target_label: "_storage_"
            replacement: "1"
            action: replace

          - regex: "(__name__)|(_storage_)|(region)|(instance)|(status)|(core)|(name)|(start_mode)|(nic)|(volume)|(state)|(version)|(mode)|(branch)|(timezone)|(goversion)|(collector)|(revision)"
            action: labelkeep
    ```
    {: codeblock}

    Where 
    
    `ENDPOINT` is the Sysdig collector endpoint. To see the list of endpoints, see [Sysdig collector endpoints](/docs/Monitoring-with-Sysdig?topic=Sysdig-endpoints#endpoints_ingestion).

    `BEARER_TOKEN` is the file that contains the **Sysdig Monitor API Token**. Notice that the file name does not have an extension.  For more information about how to get the API token, see [Getting the Sysdig API token](/docs/Monitoring-with-Sysdig?topic=Sysdig-api_token#api_token_get).

    For example, for collecting metrics through a Sysdig instance that is located in London, see the following example:

    ```yaml
    remote_write:
       - url: "https://ingest.eu-gb.monitoring.cloud.ibm.com/api/prometheus/write"
  
        bearer_token_file: C:\Users\Administrator\prom\bearer-token

        write_relabel_configs:
          - source_labels: ["__name__"]
            regex: "^wmi_(.*)"
            action: keep
    
          - source_labels: ["__name__"]
            regex: "^wmi_(.*)"
            target_label: "_storage_"
            replacement: "1"
            action: replace

          - regex: "(__name__)|(_storage_)|(region)|(instance)|(status)|(core)|(name)|(start_mode)|(nic)|(volume)|(state)|(version)|(mode)|(branch)|(timezone)|(goversion)|(collector)|(revision)"
            action: labelkeep
    ```
    {: codeblock}
    




#### Sample prometheus yaml file
{: #windows_sample_config}

```yaml
# my global config
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
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
  - job_name: 'prometheus'

    static_configs:
    - targets: ['localhost:9182']

      labels:
        instance: "window 2012 R2"
        region: "us-south"

# Connection to sysdig 
remote_write:
  - url: "https://ingest.eu-gb.monitoring.test.cloud.ibm.com/api/prometheus/write"

    bearer_token_file: C:\Users\Administrator\prom\bearer-token

    write_relabel_configs:
      - source_labels: ["__name__"]
        regex: "^wmi_(.*)"
        action: keep
    
      - source_labels: ["__name__"]
        regex: "^wmi_(.*)"
        target_label: "_storage_"
        replacement: "1"
        action: replace

      - regex: "(__name__)|(_storage_)|(region)|(instance)|(status)|(core)|(name)|(start_mode)|(nic)|(volume)|(state)|(version)|(mode)|(branch)|(timezone)|(goversion)|(collector)|(revision)"
        action: labelkeep
```
{: codeblock}

## Step 5. Monitor Windows system metrics
{: #windows_step5}

To monitor Windows systems metrics, you can use the default dashboard `Windows wmi_exporter` to view the Windows metrics. 

