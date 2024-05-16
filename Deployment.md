# Infrastructure as Code using Terraform
Create a Terraform template for creating [Ec2, Security group ] and also install docker and docker-compose using userdata method.


Create a folder called Terraform, where we store our main.tf and all other necessary files in our folder

Download keypair to the same folder

In the same folder crete a userdata.sh to install dockercontainers and docker compose

userata.sh:

```
#!/bin/bash

# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

```
![photo_6210560072392752103_y](https://github.com/shivagorasa/Prometheus-Stack/assets/97184376/0f2352be-3d12-490f-b210-b96020d0fdaf)

Main.tf :

```
# main.tf

provider "aws" {
  region = "us-east-1"
  access_key = "*******" #your aws access key goes here
  secret_key = "*******" #your aws secret key goes here
}

resource "aws_instance" "prometheus_instance" {
  ami                    = "ami-04b70fa74e45c3917" 
  instance_type          = "t2.micro"
  key_name               = "terrafdrmkey"
  subnet_id              = "subnet-05b00f52b9e78f602"
  user_data              = "${file("userdata.sh")}"
  associate_public_ip_address = true

  tags = {
    Name = "PrometheusInstance"
  }
}

resource "aws_security_group" "prometheus_sg" {
  name        = "prometheus_sg"
  description = "Security group for Prometheus instance"

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 9090 # Prometheus port
    to_port     = 9090
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  # Add any other necessary ingress rules here

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

```

Use terraform plan > terraform apply will create ec2 instance with name prometheusinstance in us-east-1 region

Ec2 instance created from terraform scripts 

![image](https://github.com/shivagorasa/Prometheus-Stack/assets/97184376/0650e604-29bf-43c6-85ca-15adab180455)

## Connect to the instance 

Multi-Container Deployments using Docker Compose
Template a docker-compose file for prometheus, Grafana and Node exporter. Apply docker-compose file and create containers.

After connecting to instance verify docker installation.

Create a folder called "Project"

create a docker-compose.yml with following to run prometheus , grafana and nodeexporter containers

```
version: '3'

services:
  prometheus:
    image: prom/prometheus
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus:/etc/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
    networks:
      - monitoring

  grafana:
    image: grafana/grafana:9.1.1
    container_name: grafana
    ports:
      - "3000:3000"
    volumes:
      - grafana_data:/var/lib/grafana
      - ./grafana/provisioning/dashboards:/etc/grafana/provisioning/dashboards
      - ./grafana/provisioning/datasources:/etc/grafana/provisioning/datasources
      - ./grafana.ini:/etc/grafana/grafana.ini  # Mount grafana.ini here
    networks:
      - monitoring

  node-exporter:
    image: prom/node-exporter
    container_name: node-exporter
    ports:
      - "9100:9100"
    networks:
      - monitoring

volumes:
  grafana_data:

networks:
  monitoring:
```
Create a folder prometheus to store prometheus.yml 

prometheus.yml:

```
global:
  scrape_interval: 15s # How frequently to scrape targets
  scrape_timeout: 10s  # How long until a scrape request times out
  evaluation_interval: 15s # How frequently to evaluate rules

scrape_configs:
  - job_name: 'node-exporter'
    static_configs:
      - targets: ['node-exporter:9100'] # The address of Node Exporter
```

with everything in place run ```docker-compose up -d``` to run containers in detached mode.

use ```docker-compose ps``` to check running process :

![image](https://github.com/shivagorasa/Prometheus-Stack/assets/97184376/d639a4bb-1ec2-4735-8653-ebf2018e88d2)


Nodeexoorter :

![image](https://github.com/shivagorasa/Prometheus-Stack/assets/97184376/a77fe6c1-e7b0-439c-bc2b-d5823eeaf1f3)

Promethues:

![image](https://github.com/shivagorasa/Prometheus-Stack/assets/97184376/64c131a8-de10-4b91-99b2-ad790c122a8d)

Grafana:

![image](https://github.com/shivagorasa/Prometheus-Stack/assets/97184376/7c311832-be7c-45bb-97f5-48fcc9089a70)

with everything runnig perfectly , Hence we created a prometheus stack with docker-compose.yml to deploy multiple services at once.


# Monitoring & Alerting using Prometheus Stack
Configure Prometheus and Grafana containers and set alerts for high cpu usage and send information using gmail.

Add Prometheus Data Source in Grafana

Access the Configuration menu: Click on the gear icon (cogwheel) in the Grafana sidebar to open the Configuration menu.

Open the Data Sources section: Navigate to the Data Sources section by clicking on it within the Configuration menu.

Add a new data source: Click on the Add data source button.

Select Prometheus data source type: From the list of available data source types, choose Prometheus.

Configure the data source settings:

URL: Enter the URL of your Prometheus server. The default URL is typically http://localhost:9090/.

Access: You can choose the access method for your Prometheus server. The options may include “Direct” or “Proxy” depending on your setup. Refer to the Grafana documentation for detailed information on access methods.

Save and test the data source: Click the Save & Test button to save the newly configured Prometheus data source and test the connection.

Once the test is successful, you can use the Prometheus data source to create dashboards and visualize metrics in Grafana.

add prometheus data sources in grafana save and test.

save prometheus data sources in grafana.

Add following pane in dashboard to monitor cpu utilization in grafana

![image](https://github.com/shivagorasa/Prometheus-Stack/assets/97184376/f5273d03-a042-4499-86a4-be88256c24c6)

Create alert for above query to get alerts in email


  Modify grafana service in docker-compose.yml
```
grafana:
    image: grafana/grafana:9.1.1
    container_name: grafana
    ports:
      - "3000:3000"
    volumes:
      - grafana_data:/var/lib/grafana
      - ./grafana/provisioning/dashboards:/etc/grafana/provisioning/dashboards
      - ./grafana/provisioning/datasources:/etc/grafana/provisioning/datasources
      - ./grafana.ini:/etc/grafana/grafana.ini  # Mount grafana.ini here
    networks:
      - monitoring

volumes:
  grafana_data:

networks:
  monitoring:
```

Make sure you have a grafana.ini file with the SMTP configuration in the same directory as your Docker Compose file.
```
[smtp]
enabled = true
host = smtp.gmail.com:587   
user = youremail@gmail.com     # use your email id 
password = yourapppassword     # use app password of google account
skip_verify = true        
from_address = youremail@gmail.com
from_name = Grafana
```

With this configuration, Grafana will use the SMTP settings provided in grafana.ini to send email notifications.

After this is done we can see following status of alerts as follows

alert active


![image](https://github.com/shivagorasa/Prometheus-Stack/assets/97184376/a7512398-10af-4d3e-9c80-320082ef837c)

save alert

![image](https://github.com/shivagorasa/Prometheus-Stack/assets/97184376/2427b5ae-6371-4c7f-b579-ff10637e6723)

to get alerts via email configure using SMTP use contact points and modify accordingly:

![image](https://github.com/shivagorasa/Prometheus-Stack/assets/97184376/9f06f492-1905-4020-9b3b-40c50a83f8f7)

when CPU utilization exceeds 80% we get following email :

![photo_6210560072392752104_y](https://github.com/shivagorasa/Prometheus-Stack/assets/97184376/79fe2eb0-ae94-4284-afaa-28557204efe4)

Thus we can successfully implemented prometheus stack and Monitoring.

___Submitted by Shiva Kumar Gorasa___


















