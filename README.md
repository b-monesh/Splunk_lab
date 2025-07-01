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

---

## 🔎 Using SPL Queries for Threat Detection
### After successfully forwarding logs from the Ubuntu machine to the Splunk server, the next step was to create custom dashboards that help visualize key security events. These dashboards are designed to assist SOC analysts in quickly identifying suspicious activities, system usage trends, and potential threats. To achieve this, I used SPL (Search Processing Language) — Splunk’s powerful query language — to filter, parse, and present relevant data. Below are some of the key queries I used, along with the types of visualizations created from them.

## 📊 Monitoring SSH Login Attempts 
### Analyze successful SSH login attempts by parsing `/var/log/auth.log` entries using Splunk. This query helps identify which IP addresses and ports are used most often for SSH access.

### 🧾 Query

#### `index=ubuntu_ source="/var/log/auth.log" "Accepted password" | rex "from (?<src_ip>\d{1,3}(?:\.\d{1,3}){3}) port (?<src_port>\d+)" | stats count as attempts by src_ip, src_port | sort -attempts`

#### 🧠 Parsed and Explained

##### 1. 🔍 `index=ubuntu_ source="/var/log/auth.log" "Accepted password"`
Searches logs from the `ubuntu_` index, specifically in `/var/log/auth.log`, and filters only lines that contain `"Accepted password"` — these indicate **successful SSH logins using passwords**.

##### 2. 🧪 `| rex "from (?<src_ip>\d{1,3}(?:\.\d{1,3}){3}) port (?<src_port>\d+)"`
Uses a regular expression to extract:

- `src_ip`: the IP address after the word **from**  
- `src_port`: the port number after the word **port**

This lets you turn raw log text into **structured fields**.

##### 3. 📈 `| stats count as attempts by src_ip, src_port`
Counts how many **successful logins** happened from each unique IP and port pair.  
The result is stored in a new field called `attempts`.

##### 4. 🔽 `| sort -attempts`
Sorts the final table in **descending order** by the number of login attempts —  
so the IPs with the most logins show up first.

### 🧪 Test Environment

#### To validate this query, SSH was set up on the Ubuntu machine, and access attempts were made from a Kali Linux system. This generated real-world authentication logs that were ingested into Splunk for analysis.

> ⚠️ The setup process is not covered here to keep the focus on the query and analysis.

![Splunk](https://github.com/b-monesh/Splunk_lab/blob/main/screenshots/14.png)

### Just like we monitored successful SSH logins, we can easily adapt the same SPL structure to detect failed login attempts, which is crucial for identifying brute-force attacks, misconfigured systems, or unauthorized access attempts 
#### SPL : `index=ubuntu_ source="/var/log/auth.log" "Failed password" | rex "from (?<src_ip>\d{1,3}(?:\.\d{1,3}){3}) port (?<src_port>\d+)" | stats count as attempts by src_ip, src_port | sort -attempts`

## 🔄 Tracking User Switching via su Command

This SPL query helps identify when users switch accounts using the `su` command. In Ubuntu systems, such activity is logged with the phrase `"session opened for user"` in `/var/log/auth.log`

### 🧾 SPL Query

#### SPL : ```index=ubuntu_ source="/var/log/auth.log" "session opened for user" | rex "session opened for user (?<user>\w+)" | stats count by user, host ```

### 🧠 Explanation

- Searches authentication logs for entries where a **new session** has been opened using the `su` command.
- Extracts the **target username** with `rex` and saves it in the `user` field.
- Uses `stats` to **count how many times each user was switched into**, grouped by `host`.

### ✅ Use Case

This gives insight into:
- Which users are being accessed via `su`
- How frequently this switching happens

### Useful for:
- 🔐 Auditing privilege usage
- 🚨 Detecting suspicious account switching

![Splunk](https://github.com/b-monesh/Splunk_lab/blob/main/screenshots/15.png)
## 👤 Monitoring New User Creation on Ubuntu

This SPL query detects when a **new user account** is created on the system by searching for `"new user"` messages in `/var/log/auth.log`.

### 🧾 SPL Query

SPL : ```index=ubuntu_ source="/var/log/auth.log" "new user"| rex field=_raw "new user: name=(?<user>\w+)"| table _time host user```
### 🧠 Explanation

- Searches for log entries that mention a **new user being added**.
- Extracts the `username` from the log using a regular expression (`name=<username>`).
- Displays a table showing:
  - `_time`: When the user was created  
  - `host`: Which machine it happened on  
  - `user`: The name of the new user

### ✅ Use Case

Helpful for:
- 🔐 Auditing user creation activity  
- 🚨 Spotting unauthorized account additions

![Splunk](https://github.com/b-monesh/Splunk_lab/blob/main/screenshots/16.png)

## 🚨 Detecting Port Scanning Attempts

#### This SPL query helps identify potential **port scanning activities** by detecting multiple port access attempts from the same source IP within a short time frame. A port scan typically involves scanning many ports on a target system, and this query highlights such behavior.

### 🧾 SPL Query

#### SPL : ` index=ubuntu_ sourcetype=syslog SRC=* DPT=* | bin _time span=1m | stats dc(DPT) as unique_ports count as attempts by _time, SRC | where unique_ports >= 10 `

### 🧠 Explanation

- Searches for **syslog entries** that include:
  - `SRC` (source IP)
  - `DPT` (destination port)
- Uses `bin _time span=1m` to group data into **1-minute intervals** for time-based analysis.
- Applies `stats` to:
  - Count unique destination ports per minute (`dc(DPT)` → `unique_ports`)
  - Count total connection attempts (`count` → `attempts`)
  - Grouped by `_time` and `SRC`
- Filters (`where`) to show only those entries where:
  - A **single source IP** accessed **10 or more unique ports** within a **1-minute window**

### ✅ Use Case

This query is useful for:

- 🕵️‍♂️ Spotting suspicious scanning activity
- 🔐 Detecting potential attackers probing for open ports
- 📊 Investigating abnormal behavior in firewall or syslog data

![Splunk](https://github.com/b-monesh/Splunk_lab/blob/main/screenshots/17.png)

### 🧪 In my case, I simulated this by scanning the Ubuntu machine using nmap from a Kali Linux machine. That’s why the logs showed a high number of unique ports being accessed — matching typical port scanning behavior.

![Splunk](https://github.com/b-monesh/Splunk_lab/blob/main/screenshots/19.png)


## 📊 Visualizing with a Dashboard

#### To make this data easier to monitor at a glance, I created a **dashboard** using the same SPL query.
#### A dashboard helps quickly spot **unusual behavior** like spikes in port scan attempts — without having to run searches manually Everytime.

### 🧩 Why Use a Dashboard?

- 📈 Real-time visibility into suspicious activity
- 🕒 Time-based charts for identifying spikes and trends
- 📋 Tables for quick review of source IPs and activity
- 🚨 Helps SOC teams act on threats faster

#### Dashboards turn raw log data into **actionable insights** using visuals like **time charts**, **bar graphs**, and **dynamic tables** — ideal for security monitoring and reporting.

![Splunk](https://github.com/b-monesh/Splunk_lab/blob/main/screenshots/18.png)

---

# 🔚 Final Thoughts
## Working on this LAB project helped me understand how logs can reveal real security events like failed logins, new user creation, and port scanning. Using Splunk and SPL gave me practical experience that connects directly to what happens in a real SOC Environment. As a cybersecurity student and enthusiast, learning SIEM tools like Splunk is helping me build a strong foundation. It made me more confident in analyzing data and thinking like a defender — which is a big step forward in my learning journey.









