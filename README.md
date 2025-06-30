# 🛡️ What is SIEM?

## SIEM (Security Information and Event Management)  is a security solution that collects, aggregates, and analyzes log data from various sources across a network in real time. It helps security teams detect threats, investigate incidents, and respond efficiently. SIEMs play a critical role in modern SOC (Security Operations Center) environments.

---


# 🔍 What is Splunk?
## Splunk is a widely used SIEM platform that collects machine data (like system logs, application logs, network activity, etc.), indexes it, and allows users to search, visualize, and analyze it. It's popular because of its scalability, powerful search capabilities (via SPL – Search Processing Language), and support for a wide range of data sources. Security teams use Splunk to monitor infrastructure, detect anomalies, and investigate incidents quickly.

---


# 🚀 Why Splunk is an Effective SIEM Tool
### 1. Real-Time Data Ingestion & Search

### 2. Alerting and Dashboards

### 3. Advanced Search and Correlation using SPL (Search Processing Language)

### 4. Supports a wide variety of data sources

### 5. Highly scalable for enterprise environments

### 6. Threat Intelligence Integration

### and many more ...

---


# 🔧 Splunk Architecture Overview
![SIEM](https://devstacks.wordpress.com/wp-content/uploads/2017/03/splunk_architecture.png)

## Splunk Components

### 1. Universal Forwarder (UF)
- A lightweight agent installed on endpoints (e.g., Windows , Ubuntu , etc..).
- Collects logs and sends them securely to the indexer.
- Ideal for minimal resource usage and continuous data streaming.

### 2. Indexer
- Receives raw data from forwarders.
- Parses, indexes, and stores the data.
- Enables fast searching and retrieval.
- May include filtering, data transformation, and enrichment.

### 3. Search Head
- Provides the web interface for users (SOC analysts).
- Allows analysts to run searches, create dashboards, alerts, and reports.

### 4. Deployment Server (Optional)
- Manages and updates configurations for multiple forwarders.

### 5. Heavy Forwarder (Optional)
- Like the Universal Forwarder but with more processing capability.
- Can perform parsing and filtering before sending to indexer.

---


# Practical SIEM Setup: Ubuntu Log Collection Using Splunk
## In this project, I set up a lab environment to simulate how a SOC analyst would use Splunk to monitor and analyze logs from an endpoint system. The focus was on collecting logs from a Ubuntu machine using the Splunk Universal Forwarder, forwarding them to a Kali machine which acts as the indexer + search_head here , and using the platform to detect and investigate security-relevant events.

## 🖧 Installing Splunk Enterprise on Kali Linux (Search Head + Indexer)
### 1. Download Splunk Enterprise
#### Go to the official Splunk download page , Sign in or create an account to get to the page shown below , click "Copy wget link" and copy the link , paste it in the terminal of the linux machine (in my case Kali) to download the .tgz file.
![Splunk](https://github.com/b-monesh/Splunk_lab/blob/main/screenshots/8.png)
### 2. Extract .tgz Splunk file 
#### `tar -xvzf splunk-9.4.3-237ebbd22314-linux-amd64.tgz`
### 3. Start Splunk and Accept License
#### `sudo splunk/bin/splunk start --accept-license`
#### Set up an admin username/password when prompted.
![Splunk](https://github.com/b-monesh/Splunk_lab/blob/main/screenshots/3.png)
#### (optional) Enable the splunk to run on boot : `sudo /opt/splunk/bin/splunk enable boot-start`
### 4.Access Web UI
#### Open browser to: http://localhost:8000

## 🖥️ Installing Splunk Universal Forwarder on Ubuntu (Log Source)
### 1. Download Splunk Universal Forwarder
![Splunk](https://github.com/b-monesh/Splunk_lab/blob/main/screenshots/9.png)

### 2. Install and start it
#### `tar -xvzf splunkforwarder-9.4.3-237ebbd22314-linux-amd64.tgz`
#### `sudo splunkforwarder/bin/splunk start --accept-license`

### Before sending logs to Indexer we need to configure something on the web UI of the Splunk Enterprise
#### - Login to the Web UI using the credential we set up earlier
![Splunk](https://github.com/b-monesh/Splunk_lab/blob/main/screenshots/10.png)
#### - In settings > indexes > New index , create a New index where all this logs from ubuntu machine gets stored , this helps to organize our logs and query it efficiently , by default these logs gets stored in main. 
![Splunk](https://github.com/b-monesh/Splunk_lab/blob/main/screenshots/11.png)
#### - In settings > Forwarding and receiving > Receive data > Add new , give a port number (ex : 9997) to listen , which makes the indexer to listen on that port and receive data from Universal Forwarders
![Splunk](https://github.com/b-monesh/Splunk_lab/blob/main/screenshots/12.png)

### 3. Configure Forwarder
#### `sudo splunkforwarder/bin/splunk add forward-server <Splunk-Server-IP>:9997 -auth username:password`
#### Add files/logs to monitor : `sudo splunkforwarder/bin/splunk add monitor /var/log -<index_name> security -auth username:password` (we can also be mention specific file from where the logs to be collected)
#### Then start the forwarder again: `sudo splunkforwarder/bin/splunk start` 

### 4. Once the Universal Forwarder is configured and sending logs from the Ubuntu machine, confirm that the data is being received and indexed by the Splunk server.
#### - Use the Search & Reporting App
#### - `index="<index_name>"`
![Splunk](https://github.com/b-monesh/Splunk_lab/blob/main/screenshots/13.png)


