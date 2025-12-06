# Project Implementation

## DEV part - Running the project locally
- Create AWS Account
- Create an EC2 instance
  - Region - London (eu-west-2)
  - Name - netflix-jenkins
  - AMI - Ubuntu
  - Instance type - t2.large
  - Create new key pair (if you don't have one)
    - name: netflix_key_pair, RSA,.pem
  - Network settings - Allow SSH, HTTP, HTTPS traffic
  - Configure storage - Size: 25 GiB
- Allocate Elastic IP to your Instance
- Add rules to the security group
  - 8081, Anywhere IPv4, app port
  - 8080, Anywhere IPv4, jenkins
  - 9000, Anywhere IPv4, sonarqube
- Connect to the Instance using EC2 Instance Connect
- sudo apt update -y
- Clone the repository on the EC2 instance
- Navigate to the repository DevSecOps-Project (you'll see the Dockerfile)
 

- In the browser, go to 'TMDB' website and create an account
- Go to your profile, create and copy the API Key


- Install Docker on your EC2 instance - check using 'docker version' command
- docker build --build-arg TMDB_V3_API_KEY=<your-api-key> -t netflix . - check using 'docker images', netflix image should be created
- docker run -d -p 8081:80 netflix
- Copy the PublicIP of the instance, in the browser search \<PublicIP>:8081 - will only open if port 8081 is open in the security group
- The Netflix clone website should be displayed successfully

## Sec part - integrating security

- Intall Sonarqube and Trivy on your instance
  - For sonarqube check using 'docker ps', 'sonarqube:lts-community' container should be displayed
    - In your browser, search \<PublicIP>:9000, sonarqube login page should be seen
    - Login using 'Username: admin' & 'Password: admin'
  - For Trivy check using 'trivy version' command
    - trivy fs . or trivy image netflix
    - You'll get a report of vulneribilities


## OPs

## CICD
  - Install jenkins on your EC2 instance
    - check using ```sudo service jenkins status```
    - In your browser, search \<PublicIP>:8080, jenkins page will be displayed, copy the path displayed
    - In your instance, ```sudo cat <path>```, copy the password and login to jenkins

  - In Jenkins, add the following 'Plugins'
    - Eclipse Temurin Installer
    - SonarQube Scanner
    - NodeJs Plugin
      install without restart
    - OWASP Dependency check
    - Docker
    - Docker Commons
    - Docker Pipeline
    - Docker API
    - docker-build-step
    - Prometheus metrics 

  - Configure the following 'Tools'
    - JDK installation
      - Name : jdk17 (should be same as the pipeline)
    - NodeJs installation
      - Name : node16 (should be same as the pipeline)
    - SonarQube Scanner installation
      - Name : sonar-scanner (should be same as the pipeline)
    - Dependency-Check installation
      - Name : DP-Check (should be same as the pipeline)
    - Docker installation
      - Name : docker

  - Create Sonarqube token
    
  - Add the following 'Credentials'
    - Add SonarQube token
      - Name : Sonar-token
    - Add Docker Username and Password
      - Name : docker
   
  - Configure the 'System'
    - SonarQube servers
      - Name : sonar-server
      - Server URL : https://<PublicIP>:9000
      - Server authentication token : Sonar-token
     
   - Delete the netflix container and image previously created
     - In your instance, ```docker ps```
     - ```docker stop netflix```
     - ```docker rm netflix```
     - check if deleted using ```docker ps```
     - Go to Docker Hub and delete the 'netflix' image
     - 
   - Add jenkins in the Docker group
     ```
     sudo su
     sudo usermod -aG docker jenkins
     ```
   - Create and run the pipeline in Jenkins
   - After the pipeline is run you can see the report in SonarQube, netflix image is created on Docker Hub, netflix container is running

## Monitoring

- Create a new EC2 instance
  - Region - London (eu-west-2)
  - Name - Monitoring
  - AMI - Ubuntu
  - Instance type - t2.medium
  - Key pair - netflix_key_pair
  - Network settings - Allow SSH, HTTPS traffic
  - Configure storage - Size: 20 GiB
- Allocate Elastic IP to your Instance
- Add rules to the security group
  - 9090, Anywhere IPv4, prometheus
  - 9100, Anywhere IPv4, node-exporter
  - 3000, Anywhere IPv4, grafana
- Run ```sudo apt update -y```
  
- Install Prometheus on your EC2 instance
- Set ownership for directories
  - ```sudo chown -R prometheus:prometheus /etc/prometheus/ /data/```
- Create a service for prometheus
  - ```sudo nano /etc/systemd/system/prometheus.service```
  - ```
    [Unit]
    Description=Prometheus
    Wants=network-online.target
    After=network-online.target

    StartLimitIntervalSec=500
    StartLimitBurst=5

    [Service]
    User=prometheus
    Group=prometheus
    Type=simple
    Restart=on-failure
    RestartSec=5s
    ExecStart=/usr/local/bin/prometheus \
      --config.file=/etc/prometheus/prometheus.yml \
      --storage.tsdb.path=/data \
      --web.console.templates=/etc/prometheus/consoles \
      --web.console.libraries=/etc/prometheus/console_libraries \
      --web.listen-address=0.0.0.0:9090 \
      --web.enable-lifecycle

    [Install]
    WantedBy=multi-user.target
    ```

 - Enable and start Prometheus
     ```
     sudo systemctl enable prometheus
     sudo systemctl start prometheus
     ```
  - Verify Prometheus's status
    ```sudo systemctl status prometheus```

  - In the browser search \<PublicIP-of-Monitoring-intance>:9090, Prometheus page should be displayed
    
  - Install Node Exporter on your instance
  - Create a service for Node Exporter
    - ```sudo nano /etc/systemd/system/node_exporter.service```
    ```
    [Unit]
    Description=Node Exporter
    Wants=network-online.target
    After=network-online.target
    
    StartLimitIntervalSec=500
    StartLimitBurst=5
    
    [Service]
    User=node_exporter
    Group=node_exporter
    Type=simple
    Restart=on-failure
    RestartSec=5s
    ExecStart=/usr/local/bin/node_exporter --collector.logind
    
    [Install]
    WantedBy=multi-user.target
    ```

  - Enable and start Node Exporter
    ```
    sudo systemctl enable node_exporter
    sudo systemctl start node_exporter
    ```
  - Verify Node Exporter's status
    ```sudo systemctl status node_exporter```
    You can access Node Exporter metrics in Prometheus

  - Add jobs to the 'prometheus.yml' file
    - Navigate to ```/etc/prometheus/```
    - Run ``` cat prometheus.yml```
      ```
      - job_name: "node_exporter"
        static_configs:
          - targets: ["<PublicIP>:9100"]

        - job_name: "jenkins"
          metrics_path: "/prometheus"
          static_configs:
            - targets: ["<PublicIP>:8080"]
      ```
    - Check syntax using
      ```promtool check config /etc/prometheus/prometheus.yml```

    - Reload Prometheus ```curl -X POST http://localhost:9090/-/reload```
    - In the browser search \<PublicIP-of-Monitoring-intance>:9090, go to 'Status' -> 'Targets', the node exporter metric should be displayed
      
    - Install Grafana on your EC2 instance
    - Enable and Start Grafana Service
      ```
      sudo systemctl enable grafana-server
      sudo systemctl start grafana-server
      ```
    - Check Grafana Status ```sudo systemctl status grafana-server```
    - In the browser search \<PublicIP-of-Monitoring-intance>:3000, Grafana login page should be displayed
    - Login using 'Username: admin' & 'Password: admin'
    - Add Prometheus as a Data source
    - Import node exporter dashboard - 1860
    - Dashboard will be displayed
    - Import Jenkins dashboard - 9964
    - Dashboard will be displayed
    


  






