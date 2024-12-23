# Promethoues-grafan-deployment-on-docker
Promethoues and grafana deployment on docker container

**SITE LINK: https://scientyficworld.org/monitor-docker-containers-prometheus-grafana/**

## Setting Up Prometheus, Grafana, and cAdvisor for Container Metrics

This document outlines the steps for setting up Prometheus, Grafana, and cAdvisor in Docker containers to collect and visualize container metrics. We will configure Prometheus to scrape cAdvisor metrics, and then display them in Grafana.

## 1. Create a prometheus.yml File

First, create a prometheus.yml file to configure Prometheus. This file will define the scrape targets, including Prometheus itself and cAdvisor.

      yaml
      Copy code
      global:
        scrape_interval: 15s  # Default scrape interval.
      
      scrape_configs:
        - job_name: 'prometheus'
          static_configs:
            - targets: ['localhost:9090']  # Default target is Prometheus itself.

## 2. Run Prometheus in a Docker Container

To deploy Prometheus in a Docker container, use the following command to pull the Prometheus image and start the container:

    bash
    Copy code
    sudo docker run -d --name prometheus -p 9090:9090 \
    -v /path/to/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus

    Replace /path/to/prometheus.yml with the actual path where you saved the prometheus.yml file.
    Once the container is running, you can access the Prometheus web interface by navigating to http://localhost:9090 in your browser. Here, you can query metrics, explore the data, and ensure that Prometheus is running correctly.

## 3. Run Grafana in a Docker Container

Next, deploy Grafana using Docker with the following command:

    bash
    Copy code
    sudo docker run -d --name grafana -p 3000:3000 grafana/grafana

Once the container is running, you can access Grafana at http://localhost:3000. The default login credentials are usually admin/admin. After logging in, change the default credentials to secure your instance.

## 4. Add Prometheus as a Data Source in Grafana

To visualize metrics from Prometheus in Grafana, you need to add Prometheus as a data source:

    Log in to Grafana at http://localhost:3000.

    Navigate to Configuration > Data Sources.
    Click Add data source and select Prometheus.
    Set the URL to http://<prometheus-container-ip>:9090, which points to your Prometheus instance.
    Click Save & Test to ensure that Grafana can communicate with Prometheus.

## 5. Run cAdvisor in a Docker Container
To collect detailed container metrics, deploy cAdvisor using the following command:

    bash
    Copy code
    sudo docker run -d --name=cadvisor -p 8080:8080 \
    --volume=/:/rootfs:ro --volume=/var/run:/var/run:ro --volume=/sys:/sys:ro \
    --volume=/sys/fs/cgroup:/sys/fs/cgroup:ro --volume=/var/lib/docker/:/var/lib/docker:ro \
    gcr.io/cadvisor/cadvisor:v0.49.2

    After starting the container, you can access cAdvisor's web interface at http://localhost:8080. Here, you will see detailed metrics about running containers.

## 6. Configure Prometheus to Scrape cAdvisor Metrics

Update your prometheus.yml file to include cAdvisor as a target:


    yaml
    Copy code
    global:
      scrape_interval: 15s  # Default scrape interval.
    
    scrape_configs:
      - job_name: 'prometheus'
        static_configs:
          - targets: ['localhost:9090']  # Default target is Prometheus itself.
    
      - job_name: 'cadvisor'
        static_configs:
          - targets: ['172.17.0.4:8080']  # cAdvisor's container IP.
    Replace 172.17.0.4 with the actual IP address of your cAdvisor container.

## 7. Reload Prometheus

After updating the configuration, you need to reload Prometheus to apply the changes. You can do this by restarting the Prometheus container:

    bash
    Copy code
    docker restart prometheus

## 8. Verify cAdvisor Metrics in Prometheus

To ensure that Prometheus is scraping metrics from cAdvisor, open the following URL in your browser:

    bash
    Copy code
    http://localhost:8080/metrics

You should see a list of metrics exposed by cAdvisor.

## 9. Query cAdvisor Metrics in Prometheus

Now that Prometheus is scraping cAdvisor metrics, you can run queries to see them. Open Prometheus at http://localhost:9090 and use the following query to display CPU usage for containers:

    prometheus
    Copy code
    sum(rate(container_cpu_usage_seconds_total{image!=""}[5m])) by (name)

    This query calculates the rate of CPU usage for each container over a 5-minute interval.

    **Get information in human readable form like MBs OR GBs**
    
    For MBs  
    sum(container_memory_usage_bytes{image!=""}) by (name) / 1024 / 1024  

    For GIBs
    sum(container_memory_usage_bytes{image!=""}) by (name) / 1024 / 1024 / 1024
  
      

## 10. Create a Dashboard in Grafana

Finally, let's create a Grafana dashboard to visualize the cAdvisor metrics.

    Log in to your Grafana instance at http://localhost:3000.

    Navigate to the Dashboards section and click on New Dashboard.
    
    Choose Add new panel to create a new graph.
    
    In the query editor, use the following PromQL query to pull data from Prometheus:
    
    prometheus
    Copy code
    sum(rate(container_cpu_usage_seconds_total{image!=""}[5m])) by (name)
    This query will show the CPU usage across all containers.

    Once you apply this query, Grafana will display a graph showing the CPU usage per container.

## 11 OR Create a Dashboard in Grafana using dashboard ID

      Dash Board id_1: 893
      Dash Board id_2: 15331

## Conclusion

You have now successfully set up Prometheus, Grafana, and cAdvisor in Docker containers to collect and visualize container metrics. Prometheus is scraping metrics from cAdvisor, and Grafana is displaying them in a dashboard for easy monitoring.


